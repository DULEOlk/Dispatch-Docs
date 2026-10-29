# UI-DISPATCH-001 旁置清单点击平移 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox syntax for tracking. 用户指定当前工作区与本会话；不自动创建工作树、提交、推送或清理文件。项目AGENTS/RULES优先于技能默认提交/清理步骤。

**Goal:** 真实点击三个交付槽后，同桌面向右平移到常驻窄清单；返回/Esc连续移回，同一卷宗和准备草稿不丢失。

**Architecture:** 保留WBP_CommissionDispatch及其展示父类，作者迁移增加固定1920×1080的等比缩放/裁剪壳和Canvas_DeskContent。卷宗原内部布局、长桌面、常驻WBP_PreparationSelection同属可移动内容。C++负责状态、槽号、输入与平移；子槽只广播点击，清单只显示空态并请求返回。

**Tech Stack:** Unreal Engine 5.6、C++/UMG/Slate、Editor Commandlet、Automation/PIE。

**Spec:** [已确认接入设计](2026-09-27-ui-dispatch-desk-pan-design.md)。

## Global Constraints

- 同一任务UI-DISPATCH-001；父级DEMO-COM-001、DEMO-VIS-001、DEMO-AUTH-001，不升级未覆盖整体项。
- 禁止物品DA/背包读取、真实条目/回填/消耗、真实派遣/存档及4B-2/4C/4D/Task5；装饰美化后置。
- 视口1920×1080；目标位移-850、时长0.85s；清单参考x1850、840×850。真实三槽几何不满足时先报告，不擅改用户水平框。
- 固定16:9；宽窄屏等比留边，不扩大可见交互范围。
- Preserve原NewWidgetBlueprint，不重命名或删除，不产生redirector；替换运行引用而非覆盖旧素材。
- 资产先精确备份与记录哈希，写入前复核编辑器进程；不SaveAll，不碰其他脏改/未跟踪资产，不提交/推送。
- 每个RED/实施/GREEN/阻塞阶段同步STATUS、IMPLEMENTATION_MATRIX及现有handoff。

## Review Focus

1. 动画途中Esc/重复打开：任务2测反向连续、拒绝重入及当前槽不被改写。
2. 子按钮吃掉键盘/焦点：任务3真实Slate/PIE路由Esc、返回后焦点可继续操作。
3. 非16:9/DPI及窗口改变：任务3检查裁剪、完整三槽和清单边界，不能只测默认分辨率。
4. 关闭PIE/重新初始化时仍在动画：任务2/3测取消、回初始、无残留页面和焦点抢占。
5. 作者迁移遇到未知层级或同名资产：任务1先probe，非预期状态拒绝写入，失败报告已保存目标/备份，不覆盖或自动删除恢复。

## 命令与证据

项目 `F:/Epic Games/Unreal Projects/Dispatch/Dispatch.uproject`。

```powershell
& 'F:/Epic Games/games/UE_5.6/Engine/Build/BatchFiles/Build.bat' DispatchEditor Win64 Development '-Project=F:/Epic Games/Unreal Projects/Dispatch/Dispatch.uproject' -WaitMutex -FromMsBuild
& 'F:/Epic Games/games/UE_5.6/Engine/Binaries/Win64/UnrealEditor-Cmd.exe' 'F:/Epic Games/Unreal Projects/Dispatch/Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Widget.CommissionDispatch;Quit' -RenderOffscreen -unattended -nop4 -nosplash -DDC-ForceMemoryCache '-abslog=F:/Epic Games/Unreal Projects/Dispatch/Saved/Logs/DeskPan-Dispatch.log'
```

其他过滤器用同一命令单独替换：`Dispatch.Widget.PreparationSlot`、`Dispatch.Editor.StoryWorkspace.PreviewCoordinator.Commission`；最终`Dispatch.Widget`回归。RED/GREEN日志用阶段后缀避免覆盖；图证据`Saved/Automation/CommissionDeskPan/`。统计Test Completed与失败名称，不把进程退出当作通过；需要权限时正常申请，不绕引擎锁。

## Task 1：作者层级与空清单

**Files:** 新增 `Source/DispatchEditor/Public/CommissionDeskPanCommandlet.h`、`Private/CommissionDeskPanCommandlet.cpp`；新增 `Source/Dispatch/Public/Widget/PreparationSelectionWidgetBase.h`及对应Private实现；新增 `Source/DispatchEditor/Private/Tests/CommissionDeskPanTests.cpp`；修改目标卷宗uasset、新建清单及纹理。

**Interfaces:** `UCommissionDeskPanCommandlet::Main(const FString&)`支持默认只读probe及显式`-Apply`；`UPreparationSelectionWidgetBase::SetSelectedSlotIndex(int32)`更新1/2/3，原生无参数返回delegate。控件名 `Canvas_DeskContent`、`Size_DeskViewport`、`Scale_DeskViewport`、`PreparationSelection`；清单 `Text_SelectedPreparationSlot`、`Scroll_Preparations`、`Text_EmptyState`、`Button_ReturnToDossier`。

- [ ] 先写 `Dispatch.Widget.CommissionDispatch.DeskLayout`：加载实际蓝图，新裁剪层/等比壳/常驻清单存在；空列表0项、空态可见、中文标题和返回；原Box_PreparationSlots及其三个独立子Widget仍存在。构建运行取得因缺少新结构的RED（不是夹具或编译拼写失败）。
- [ ] 实现最小清单基类和独立作者工具；probe输出完整目标层级、原各CanvasSlot数据、新旧资产是否存在和计划目标。原卷宗子树作为整体被包裹，除旧背景的指定引用/绘制区域外，其内部槽位布局/文本/图片不改。
- [ ] 将已确认木桌PNG复制到新的项目源图路径 `Content/UI/Commission/Background/T_DispatchDesk_Pan.png`；导入同名新纹理，不覆盖002。新建 `/Game/BluePrints/Widget/Menu/Quest/WBP_PreparationSelection`；保留旧概念页。
- [ ] 写入前备份目标uasset与manifest到`Saved/CodexBackups/CommissionDeskPan_<timestamp>/`；只保存本任务目标包。编译失败/结构不符终止，不继续写其他包；重复Apply不得重复包裹或新增清单。
- [ ] 新进程重载执行DeskLayout和PreparationSlotRow，确认通过与排版manifest一致；记录实际资产diff/引用。此时不宣称交互完成。

## Task 2：三槽点击与可逆平移

**Files:** 修改 `Source/Dispatch/Public/Widget/PreparationSlotWidgetBase.h`及Private实现、`CommissionDispatchUIBase.h`及Private实现；修改 `Source/Dispatch/Private/Tests/CommissionDispatchPresentationTests.cpp`、编辑器 `PreparationSlotWidgetTests.cpp`和新`CommissionDeskPanTests.cpp`。

**Interfaces:** 槽新增 `void SetSlotIndex(int32)` 与 `FOnPreparationSlotClickedNative(int32)`；仅合法0～2广播。卷宗保留 `bool OpenPreparationPageForSlot(int32)`、`void ClosePreparationPage()`、`bool IsPreparationPageOpen() const`、`UUserWidget* GetPreparationPage() const`。常驻实例与打开状态分离；内部四态及位移只驱动作者Canvas的RenderTranslation，NativeTick推进；不暴露仅供测试的生产API，测试通过实际树/几何/输入观察行为。

- [ ] 写 `DeskPanInteraction`、`DeskPanReversal`、`DeskPanLifecycle` 与槽编号/点击测试；旧代码运行取得RED：点击新三槽不驱动平移，或仍创建独立页。测试先持有Slate树并完成布局，避免此前测试夹具的prepass问题。
- [ ] 槽广播真实Button的OnClicked；重复Construct不重复绑定，Destruct解绑；编号显示1/2/3，hover/press实现保持。
- [ ] 卷宗按Box_PreparationSlots顺序给三实例编号并绑定；BindWidgetOptional读取新容器以兼容无作者树的纯数据测试。缺少结构时Open返回false且不改槽选择/草稿，不退回旧弹出方案。
- [ ] 用进度p与smoothstep `p*p*(3-2*p)`驱动0至-850，恒定进度速度1/0.85。Close反向当前p，无瞬跳；重复Open拒绝，MovingToDossier拒绝Open；IsPreparationPageOpen在进入/清单态true，退出态false但仍由内部状态屏蔽重入。
- [ ] 只更新清单槽号，不重建/隐藏整份卷宗。输入屏蔽卷宗内容但不灰化；清单端返回聚焦，退出完成还原当前槽焦点。Esc在进入/清单态返回，卷宗态不吞无关退出；NativeDestruct只取消/解绑，不调用会抢焦点的交互关闭流程。
- [ ] InitializePreviewDraft与重开复位导航；RequestReturn/RequestPreviewDispatch不得从清单或移动中误触发。前后深度比较既有草稿与Presentation不变；范围外ChoosePreviewPreparation仍不接UI。
- [ ] 更新AuthoredBindings旧透明覆盖按钮断言为三个真实子Widget的有效点击链（不是删除断言）；运行全部相关Widget过滤器，记录RED→GREEN与既有失败。

## Task 3：PIE、分辨率与实际窗口验收

**Files:** 修改 `Source/DispatchEditor/Private/Tests/StoryUIPreviewCoordinatorTests.cpp`及`CommissionDeskPanTests.cpp`；`StoryWorkspace/StoryUIPreviewCoordinator.cpp`仅确有生命周期接缝时修改。

**Interfaces:** 继续现有Coordinator创建/返回/停止接口；自动化夹具通过正式作者Widget启动导航，不要求无树的UCommissionDispatchUIBase具备平移画面。保留纯数据草稿验证，不往生产类塞测试路径。

- [ ] 先更新旧PIE断言为“相同常驻子页、不单独IsInViewport、同卷宗始终存在”，并用latent等待动画终点；运行失败确认旧预期/新状态接缝，再做最小适配。
- [ ] 真实鼠标路由命中三个Button，返回按钮及键盘Esc；当前委托、NPC与有内容的草稿前后相同；正向/反向中间帧连续，连续点击无叠页。检查结束PIE/重开仍从卷宗端开始，delegate不重复。
- [ ] 渲染1920×1080、2560×1440、2560×1080、1280×960；实际几何验证初始只露纸边、交付端全部三槽及完整清单在裁剪框内，图像/文字未拉伸。若真实卷宗位置导致三槽被裁，不为测试静默改用户排版，报告需要调整的外层参数。
- [ ] 用现有测试Commission/NPC预览验证CommissionSubsystem实例不新增，库存/存档无改变；不得修改源DA去制造测试条件。
- [ ] 运行CommissionDispatch、PreparationSlot、Coordinator Commission聚焦及Dispatch.Widget宽回归，准确记录每项失败。旧HoverDimsPeers、IdleDimOption、CommissionUIBase.Behavior若仍RED则列名，不扩范围修复或冒称全套通过。
- [ ] 实际窗口手点三槽、返回/Esc、快速重复与关闭重开；有可用原生控制能力才声称人工执行，工具受限则交用户验收并标待验收。
- [ ] 本会话完成范围内review，按项目要求更新STATUS/矩阵/handoff与资产manifest；需要独立review时仅按技能授权派发，不要求用户接管新任务；无提交、推送或清理。

## 自检与当前停止点

三项分别覆盖作者结构、运行导航和集成验收，全部范围可追溯到已确认Spec。五项Review Focus均有测试归属；不包含物品业务或美化。计划尚未执行，RED/资产备份/源码改动均未发生。请用户确认这份执行清单；推荐当前会话串行执行，不切新工作树，以保留当前脏资产为基线。执行记录继续用本任务handoff，不删除本地docs。
