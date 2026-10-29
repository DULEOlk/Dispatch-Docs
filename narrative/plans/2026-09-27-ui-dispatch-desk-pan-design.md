# UI-DISPATCH-001 旁置交付清单平移接入设计

日期：2026-09-27。状态：用户已回复“可以”确认技术接入设计；生产实现未开始。

## 目标与边界

用户已认可独立预览的“卷宗旁放一张窄清单”空间关系，最新要求现在实现点击后的视角移动，装饰和美化以后再做。本次沿用 UI-DISPATCH-001，父级 DEMO-COM-001、DEMO-VIS-001、DEMO-AUTH-001；不新开并行主要任务。

只实现展示和导航：三槽分别传0/1/2、同桌面平移、空清单、返回/Esc、状态保持、固定比例裁剪。没有真实物品条目、选择/回填、数量/互斥/消耗、真实派遣或存档，不实施4B-2/4C/4D/Task5。不把内部预览草稿已有ChoosePreviewPreparation接口接到清单。

## 已核对事实

- 当前 CommissionDispatchUIBase::OpenPreparationPageForSlot 仍同步加载 NewWidgetBlueprint，CreateWidget、隐藏卷宗、AddToViewport(100)。ClosePreparationPage 则RemoveFromParent并恢复卷宗。这条旧路径必须替换，不能在其上只加位置动画。
- EnsurePreparationSlotButtons仍寻找旧Image_PreparationSlot1/2/3Frame；现有水平框已经换为独立WBP_PreparationSlot，故旧点击绑定未恢复。
- PreparationSlotWidgetBase目前仅悬浮/按下外观，不广播点击。用户已有柔和反馈须保留。
- StoryUIPreviewCoordinator仍用同一个CommissionDispatchUIBase实例，InitializeDraft后AddToViewport。当前PIE测试断言交付页IsInViewport/关闭移除，属于已被新设计替代的旧合同。
- 本轮检查未发现UnrealEditor进程；实际资产写入前仍须再次检查。当前main，HEAD 11939c7f0d0ed9a63f47eb5f2115f2d62e6ff426。源码/蓝图/删除记录和未跟踪文件全部保留。

## 方案

### 1. 同一个Widget内的桌面

保留WBP_CommissionDispatch资产路径与C++父类，保留原卷宗所有内容和水平框的内部排版。在外层增加固定16:9的等比缩放与裁剪容器；卷宗内容、长桌面和旁置清单位于同一个可平移Canvas中。桌面移动，外部裁剪窗口不动。不是3D摄像机，不控制玩家Pawn/关卡镜头。

逻辑取景1920×1080；初始平移0，交付端初始目标-850，850ms平滑插值。根据当前真实卷宗几何校准清单位置，优先保留用户布局，不照搬截图裁切尺寸去缩放整份卷宗。清单参考x1850、840×850；初始只露少量纸边/标签，交付端完整清单和三槽必须都可见。若当前几何与参考不符，作者工具报告不符合项，不静默改内部排版。

16:9不同分辨率只改变等比缩放。21:9外侧留暗色桌面/底色，不扩大内部取景；4:3等比缩放留边。不做新增装饰。新桌面纹理使用已确认的压暗左侧版本并新建命名资产，不覆盖原002纹理，原引用可由备份恢复。

### 2. 常驻空清单

新建WBP_PreparationSelection及最小展示基类，标题、槽位提示、空列表容器、空状态、返回按钮都在作者Widget内。纸面用普通占位底色即可；不再生成正式美术。

它作为卷宗内常驻子Widget，在卷宗构建时一次创建；点击只更新槽号与动画目标，不CreateWidget/AddToViewport。关闭只移回并恢复输入，不销毁子Widget。保留原NewWidgetBlueprint，不重命名、不删资产，不产生重定向；运行时新路径不再依赖旧概念页面。

### 3. 三槽接线与状态机

独立槽只广播自身编号的点击事件，由CommissionDispatchUIBase负责导航，槽控件不接业务系统。三个实例编号0/1/2，显示1/2/3；绑定可重复构建安全，析构时解绑，保留现有hover/press动画。

导航状态为Dossier、MovingToPreparation、Preparation、MovingToDossier。打开后立即保存当前槽号；已打开/正打开时重复点击不叠页、不重新开始动画、不改变当前槽。返回/Esc可在进入动画途中反向，从当前位置连续返回；返回途中拒绝新打开，避免竞态。

只在卷宗端允许槽按钮输入；清单端焦点在返回按钮，外侧卷宗仅作参照，不触发派遣/返回委托选择。关闭清单不是RequestReturn（后者退出卷宗回委托选择）。输入屏蔽不整体Disabled卷宗，以免其颜色被灰化。原有委托、NPC、三槽草稿与展示图不重新初始化；GetPreparationPage可返回常驻实例，IsPreparationPageOpen必须独立于对象存在性判断。

销毁/退出预览时取消动画、解绑，不抢回已销毁界面的焦点，不残留子页到viewport；重新打开从卷宗端开始。若重置预览草稿时清单处于打开状态，也同步重置导航。

## 修改范围与保护

- CommissionDispatchUIBase.h/.cpp：导航状态、NativeTick/键盘/焦点、常驻子页与新槽绑定，移除旧弹出路径。
- PreparationSlotWidgetBase.h/.cpp：编号和展示层点击广播，保留外观处理。
- 新PreparationSelectionWidgetBase.h/.cpp：槽提示、返回请求，无物品数据。
- WBP_CommissionDispatch：新增外部容器和旁置清单，不覆盖内部卷宗排版；WBP_PreparationSlot只在必要时更新编号绑定。
- 新WBP_PreparationSelection和新长桌面纹理。作者迁移工具使用独立文件，避免继续扩张现有CommissionCardEffectsCommandlet。
- CommissionDispatchPresentationTests、PreparationSlotWidgetTests、StoryUIPreviewCoordinatorTests及新平移测试；Coordinator仅在生命周期/输入确需适配时修改。

资产写入前备份本轮精确目标到Saved/CodexBackups，保存原路径/哈希/层级及Canvas槽信息；拒绝覆盖未知已存在的新命名资产。不开SaveAll，不动PlayerNotebook、旧Inventory_UI删除、正式DA、Config/地图或其他用户素材。继续当前用户工作区，不复制不含未提交资产的“干净”工作树，不提交/推送。

## 验证与退出条件

先写实际行为断言并取得RED，再实施：

1. 新三槽真实按钮命中/点击分别得到0/1/2，现有hover/press回归不退化。
2. 同一卷宗和同一常驻清单；卷宗不Collapsed，清单不单独IsInViewport；重复点击页面计数不变。
3. 动画初始、中间、终点都有实际Transform变化；850ms到目标；中途Esc反向无位置跳变，返回0；再次打开不残留状态。
4. 返回按钮/Esc真正路由到关闭；焦点恢复，隐藏/移出区域不可点击；析构/结束PIE时没有孤立页面和错误焦点。
5. 1920×1080、2560×1440、2560×1080、1280×960的真实Slate几何和截图验证固定16:9、露边量、清单完整及卷宗三槽保留。
6. 前后委托/NPC展示、三个草稿深度比较不变，CommissionSubsystem实例数量/库存/存档无增量。

聚焦构建、Widget与相关Coordinator测试；扩大回归按项目RULES适度执行。旧AuthoredBindings不删测试来掩盖失败，改为检查新三实例真实接线。其余既有HoverDimsPeers、IdleDimOption、CommissionUIBase.Behavior三项范围外失败如仍出现则逐项报告。

最后在实际预览窗口手点三个槽，观察来回移动、Esc和快速重复；自动截图不冒充人工验收。若无法控制原生窗口，则明确人工验收待用户完成，不写通过。文档三处同步每个可验证小步。

## 当前门槛

技术设计已获用户确认。执行清单见同目录 `2026-09-27-ui-dispatch-desk-pan-implementation.md`；生产源码/蓝图尚未修改，没有新的UE RED或GREEN。
