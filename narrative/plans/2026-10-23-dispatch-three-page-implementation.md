# 派遣卷宗三形态拆分 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. 当前方案推荐本会话直接实施；未经用户明确选择不派生代理。

**Goal:** 将现有右页拆成三个可独立设计的Widget，保留左页与原派遣前交互。

**Architecture:** 原WBP_CommissionDispatch保留公共外壳、左页和桌面导航，WidgetSwitcher切换三份右页。轻量子页基类提供命名控件与展示入口；现有草稿和交互仍由父页管理，查找/创建控件改用准备页的WidgetTree。

**Tech Stack:** UE5.6、C++/UMG、Widget Blueprint、编辑器资产迁移命令、真实Slate/RHI自动化。

**Spec:** [已确认结构设计](2026-10-23-dispatch-three-page-design.md)

## Global Constraints

- UI-DISPATCH-001，父级DEMO-COM-001 / DEMO-VIS-001 / DEMO-AUTH-001；报告DEMO-COM-002业务未覆盖。
- 左页、公共纸面/装订、桌面和返回保留；只迁移右页专属装饰与内容。
- 三页名称固定WBP_DispatchPage_Preparation / InProgress / Completed，置于Quest/DispatchPages。
- 派遣前外观与既有准备槽/蜡封/旁置清单交互保持。另两页仅独立Canvas和命名容器，占位布局由用户后续设计。
- 展示模式不等于正式事务状态；首次报告NPC规则保持，不接标签打开页、结果、奖励、正式DA或存档。
- 已有其他Widget修改、Inventory/PNG库素材删除和六件指定保留图标均保持；不提交/推送，不强制加入忽略的docs。

## Review Focus

1. Designer预览和真实实例的生命周期不同：默认与手动切换都必须显示正确子页。
2. 原父页名称查找不能穿过子Widget树：NPC、准备槽、动态蜡封层都需查找/创建在正确树内。
3. 从打开的准备清单切到只读页：复位桌面、隐藏清单、释放失效焦点，拒绝继续交付。
4. 重开或反复切换：不能重复订阅槽点击、残留蜡封悬浮层或清掉作者预览文字。
5. 缺省/空只读布局：父页左侧正常、返回可用，不读取或显示任何隐藏报告内容。

## 文件与接口分工

- 新建 `Source/Dispatch/Public/Widget/CommissionDispatchPageBase.h`、`Private/Widget/CommissionDispatchPageBase.cpp`：子页可选NPC绑定与访问入口，不保存草稿。
- 修改 `Source/Dispatch/Public/Widget/CommissionDispatchUIBase.h`、`Private/Widget/CommissionDispatchUIBase.cpp`：定义 `ECommissionDispatchPageMode {Preparation, InProgress, Completed}`；父页调度与旧绑定迁移。
- 新建 `Source/DispatchEditor/Private/CommissionDispatchPagesSetup.cpp`：可重复识别完成状态、遇不明结构拒绝的资产迁移；接入已有CommissionCardEffectsCommandlet的 `-SplitDispatchPages` 参数。
- 新建 `Source/DispatchEditor/Private/Tests/CommissionDispatchPageTests.cpp`：结构/三态/生命周期。
- 修改 `Source/DispatchEditor/Private/Tests/CommissionDeskPanTests.cpp`：只调整移入子树的定位方式，保留真实命中和数值/行为断言。
- 修改现有WBP_CommissionDispatch，新增三份子Widget；其他资产不保存。

### Task 1：三态展示与隔离接口

**Interfaces:** 父页公开 `void SetPageMode(ECommissionDispatchPageMode Mode)`、`ECommissionDispatchPageMode GetPageMode() const`、`UCommissionDispatchPageBase* GetPage(ECommissionDispatchPageMode Mode) const`；编辑器属性 `PageMode` 默认Preparation。子页提供 `void ApplyNPCPresentation(const FCommissionDispatchPresentation& Data)`、`UWidget* FindPageWidget(FName Name) const`，NPC绑定为Optional。父页提供内部 `UWidget* FindPreparationWidget(FName Name) const`，仅定位准备子页。

- [ ] 新增自动化 `Dispatch.Widget.CommissionDispatch.PageModes`：三种值切换/无效枚举不改变当前页、同值幂等、切换前后左页展示不变；缺少可选NPC控件不崩溃。
- [ ] 构建测试取得缺接口红灯证据，再最小实现上列接口与可选子页绑定；父页原左页BindWidget保留，五个右页NPC字段迁至子页。
- [ ] NativePreConstruct/Construct均应用模式；只有显式收到Presentation才覆写子页作者文字。ApplyPresentation把NPC数据交给子页；不新增结果数据读取。
- [ ] 非Preparation时SelectPreparationSlot / OpenPreparationPageForSlot / ChoosePreviewPreparation / RequestPreviewDispatch返回false。离开Preparation调用现有桌面复位和关闭清单；焦点仅在确实属于消失区域时移回父页。RequestReturn仍有效。
- [ ] 构建通过；模式行为测试运行到旧资产缺少子页/切换器的预期失败，不能把夹具失败当作业务红灯。

### Task 2：安全迁移三份可编辑资产

**Interfaces:** 迁移入口 `int32 SplitCommissionDispatchPages()`；父页绑定 `Switcher_DispatchPages` 与子实例 `Page_Preparation` / `Page_InProgress` / `Page_Completed`。

- [ ] 新增 `Dispatch.Widget.CommissionDispatch.PageAssets`：三份独立WidgetBlueprint类、统一子页基类、父Switcher顺序Preparation/InProgress/Completed、默认Preparation；旧资产运行取得明确缺结构断言。
- [ ] 执行前审计当前父树与右页装饰，记录每件迁移控件的资源/文本/屏幕坐标/层级和绑定。将现行父Widget备份到带时间戳的Saved/CodexBackups；检查三新路径未占用。
- [ ] 右页局部区域原点(852,80)，尺寸(740,960)；Switcher在原Canvas_Root内按此矩形布局。将NPC、准备槽、准备标题/反应框、共用上下线、派遣按钮及右页专属纹饰迁入Preparation，转换为局部坐标，保留原屏幕几何。公共纸面、左页纹饰和返回留父页。迁移表必须以当时资产实测为准；不明或越界内容停下检查，不猜测删除。
- [ ] InProgress / Completed分别建Canvas_Root、Canvas_NPCInfo、Canvas_Content、Canvas_Actions四个命名容器，匹配同一740×960区域；不生成假报告/奖励内容，不预设用户排版。
- [ ] 父页涉及旧命名字段移除时同步OnVariableRemoved，子页新增字段同步变量记录；编译三子页再编译父页。只保存四份Widget，原素材/源图不变。
- [ ] 将原蜡封控件创建/渐变层归入准备页WidgetTree，保持原UV与原相对尺寸、无业务点击绑定；父页Tick查找准备子树中的层。准备槽仍复用原WBP_PreparationSlot和单一父草稿。
- [ ] PageAssets通过；准备页能单独打开编辑，三子页不共享WidgetTree对象，重复作者命令验证完成状态并退出，不覆盖用户后续布局。

### Task 3：真实交互与交接

**Interfaces:** 沿用Task1公开GetPage/FindPageWidget供测试定位，不添加穿透任意子树的全局搜索兼容层。

- [ ] 将既有测试里确实迁入准备子页的控件查找改为上述接口，保持原派遣槽/蜡封行为断言；局部位置断言换算回屏幕/父页坐标，不能仅删除旧断言。
- [ ] 新增 `Dispatch.Widget.CommissionDispatch.PageLifecycle`：打开准备清单中切到InProgress，清单关闭/导航复位/继续选择拒绝；Preparation→Completed→Preparation重复3轮，单次槽点击只有一次响应，重建Widget后默认/指定模式一致。只读空页左侧与返回正常，切换不发派遣确认事件。
- [ ] 跑PageModes / PageAssets / PageLifecycle与现有WaxButtonStates / DeskPanInteraction / DeskPanReversalAndLifecycle / DeskPanRenderedGeometry。若历史测试名变动先定位实际注册名；不要用不存在过滤器的零测试成功充数。
- [ ] 真实RHI核对1920×1080、2560×1440、2560×1080、1280×960的Preparation屏幕几何/命中保持，并输出InProgress/Completed占位截图。用户后续自行排版，空容器不得误报为完成设计。
- [ ] 更新CURRENT、矩阵UI-DISPATCH-001和阶段handoff，分别报告结构已拆、构建/局部测试结果、Designer人工验收未做。父级完整功能仍需要重验；现存准备清单旧失败单列，不强行纳入本次修复。

## 验证命令与证据

PowerShell构建（Development，项目原路径）：

```powershell
& 'F:/Epic Games/games/UE_5.6/Engine/Build/BatchFiles/Build.bat' DispatchEditor Win64 Development '-Project=F:/Epic Games/Unreal Projects/Dispatch/Dispatch.uproject' -WaitMutex -NoHotReloadFromIDE -NoUBA
```

作者命令：UnrealEditor-Cmd.exe + 同一uproject + `-run=CommissionCardEffects -SplitDispatchPages -AllowCommandletRendering -RenderOffscreen -unattended -nop4 -ddc=InstalledNoZenLocalFallback`。先设UE-LocalDataCachePath到项目Saved/CodexDDC；构建/作者/每轮自动化分别记录独立Saved/Logs/DispatchThreePages*日志。

自动化同一exe/uproject，`-ExecCmds="Automation RunTests Dispatch.Widget.CommissionDispatch.PageModes+Dispatch.Widget.CommissionDispatch.PageAssets+Dispatch.Widget.CommissionDispatch.PageLifecycle+Dispatch.Widget.CommissionDispatch.WaxButtonStates+Dispatch.Widget.CommissionDispatch.DeskPanInteraction+Dispatch.Widget.CommissionDispatch.DeskPanReversalAndLifecycle+Dispatch.Widget.CommissionDispatch.DeskPanRenderedGeometry;Quit" -RenderOffscreen -unattended -nop4 -ddc=InstalledNoZenLocalFallback`。

成功要求：实际Test Completed逐项成功、Controller错误0、最终进程退出0；资产保存无Blueprint Error/ensure；实际截图复看。静态结构通过不能代替真实交互通过。

## 计划自检与实施门槛

已对照设计逐项覆盖资产三分、局部坐标/共同左页、准备交互迁移、Designer切换、生命周期、未听报告保密与工作区保护；Review Focus五项分别由Task1/Task3行为测试和Task2资产测试覆盖。无新增正式DA字段、无提交步骤、无代理任务。

当前仅计划完成，未修改生产源码/Widget、未运行新的实现测试。用户审阅计划并选择执行方式后再实施；推荐本会话直接实施，因为三个步骤共享同一父Widget与绑定迁移，顺序依赖强。

## 2026-10-23 执行结果（最新）
三任务实施及最终审查已完成，现有派遣前右页迁移，另外两页仅空容器；未接标签。FinalBuild成功、FinalVerify20/20、Controller错误0、退出0；作者与幂等检查成功，保护资产哈希变化0。原生Designer/真实玩家焦点等待人工验收。测试订阅计数断言力度Minor已记录；未确认重复触发缺陷，不增加本轮范围。详细失败→修复与证据见handoffs/2026-10-23-dispatch-three-pages.md。不提交/推送。
