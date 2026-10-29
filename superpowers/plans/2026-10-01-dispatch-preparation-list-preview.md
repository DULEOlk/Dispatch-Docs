# Dispatch Preparation List Preview Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在现有交付清单中实现可点击预览列表、选中详情及放入当前准备格，并给用户提供 UE 实际截图/点击验收入口。

**Architecture:** `UCommissionDispatchUIBase` 继续拥有预览数组和三格草稿；`UPreparationSelectionWidgetBase` 只呈现列表/详情并发出所选索引，绝不访问库存或派遣子系统。现有 `WBP_PreparationSelection` 保持底图/按钮槽，把真实 UMG 行与详情控件放在纸面上；预览会话传入的 `FCommissionPreparationViewData` 是唯一列表输入。

**Tech Stack:** UE 5.6 C++、UMG Widget Blueprint、DispatchEditor 自动化与 WidgetRenderer。

**Spec:** `docs/superpowers/specs/2026-10-01-dispatch-preparation-list-preview-design.md`

## Global Constraints

- 保留当前底图、阴影、桌面平移、卷宗、返回/Esc 语义。
- 不读取/消耗正式背包，不写正式 DataAsset、存档或真实派遣事务；不烘入条目/文字/状态。
- 用户未授权提交/推送；本地 `docs/` 被忽略，保留所有已有脏改，Widget 保存前逐文件备份。
- 唯一实施子任务 `UI-DISPATCH-001`，父级 `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`；不宣布父级整体完成。

## Review Focus

- 空输入：保留“暂无交付物数据”，确认禁用（Task 1 测试）。
- 同一物品已放在另一格：不允许再次提交并明确显示不可选（Task 2 测试）。
- 已填格重新打开：预选当前物并允许换成另一个，其他格不受影响（Task 2 测试）。
- 返回/Esc 与平移中关闭：只退出页面，不提交候选；焦点恢复（Task 2 测试）。
- 超过四件物品、长名称/无图标：列表滚动、文本裁切和详情占位不溢出纸面（Task 3 渲染测试）。

---

### Task 1: 交付页列表与详情呈现

**Files:**
- Modify: `Source/Dispatch/Public/Widget/PreparationSelectionWidgetBase.h`
- Modify: `Source/Dispatch/Private/Widget/PreparationSelectionWidgetBase.cpp`
- Modify: `Content/BluePrints/Widget/Menu/Quest/WBP_PreparationSelection.uasset`（仅 Widget 控件，先备份）
- Modify: `Source/DispatchEditor/Private/CommissionDeskPanCommandlet.cpp`（一次性、状态防护的 Widget 作者命令）
- Test: `Source/DispatchEditor/Private/Tests/CommissionDeskPanTests.cpp`

**Interfaces:**
- Consumes: `FCommissionPreparationViewData` 的 `Preparation/DisplayName/Description/Icon/AvailableCount`。
- Produces: `void SetAvailablePreparations(const TArray<FCommissionPreparationViewData>& InPreparations, int32 CurrentSelectionIndex, const TSet<int32>& UnavailableIndices)`；`int32 GetHighlightedPreparationIndex() const`；`FOnPreparationCommitRequestedNative(int32)`。动态行按钮名 `Button_PreparationEntry_<Index>`；详情控件名 `Image_DetailIcon`、`Text_DetailName`、`Text_DetailType`、`Text_DetailDescription`，反馈名 `Text_SelectionMessage`。

- [ ] **Step 1: 写 RED 测试**：`Dispatch.Widget.CommissionDispatch.PreparationListPresentation`，断言空态/禁用；注入六项后 `Scroll_Preparations` 六行、可滚动，点击 `Button_PreparationEntry_2` 后 `GetHighlightedPreparationIndex()==2`、详情名称/描述/图标和单一高亮同步；无图标显示占位，长文案不出布局边界。
- [ ] **Step 2: 构建并运行该测试，确认仅因缺失呈现/控件而 RED**：`Build.bat DispatchEditor Win64 Development -Project=<Dispatch.uproject> -WaitMutex -NoHotReload`，`UnrealEditor-Cmd.exe <Dispatch.uproject> -ExecCmds="Automation RunTests Dispatch.Widget.CommissionDispatch.PreparationListPresentation;Quit" -unattended -nop4 -DDC-ForceMemoryCache`。
- [ ] **Step 3: 实现呈现**：作者命令只增加纸面条目行承载及详情控件；基类动态创建行，按钮、图标、名称、摘要、状态均为 UMG，维护唯一 `HighlightedPreparationIndex`，设置详情和确认可用性；底图/按钮槽/原有返回与确认坐标不改。
- [ ] **Step 4: 同一过滤器 GREEN，检查保存前后 Widget 默认状态与空态，保留资产备份**。

### Task 2: 当前格提交与卷宗回填

**Files:**
- Modify: `Source/Dispatch/Public/Widget/CommissionDispatchUIBase.h`
- Modify: `Source/Dispatch/Private/Widget/CommissionDispatchUIBase.cpp`
- Modify: `Source/Dispatch/Public/Widget/PreparationSlotWidgetBase.h`
- Modify: `Source/Dispatch/Private/Widget/PreparationSlotWidgetBase.cpp`
- Test: `Source/DispatchEditor/Private/Tests/CommissionDeskPanTests.cpp`
- Test: `Source/DispatchEditor/Private/Tests/StoryUIPreviewCoordinatorTests.cpp`

**Interfaces:**
- Consumes: Task 1 的 `SetAvailablePreparations` 与提交索引事件；现有 `ChoosePreviewPreparation(int32)`、`PreviewDraftSlots`。
- Produces: `void UPreparationSlotWidgetBase::SetPreviewPreparation(const FCommissionPreparationViewData* Data)`；`void UCommissionDispatchUIBase::HandlePreparationCommitRequested(int32 PreparationIndex)`。

- [ ] **Step 1: 写 RED 测试**：`Dispatch.Widget.CommissionDispatch.PreparationCommitFlow`，覆盖三格分别放入、同物跨格拒绝、重开预选/改选、返回/Esc 不提交、完成后卷宗格显示对应名称；Coordinator 测试确认预览结果只含提交过的格位。
- [ ] **Step 2: 构建并运行两个过滤器，确认失败点是未接事件或未回填显示**。
- [ ] **Step 3: 最小接线**：打开页面时传入当前可用数组、当前格已有选择索引及其他格占用索引；收到确认事件后调用 `ChoosePreviewPreparation`，成功才更新三格显示并 `ClosePreparationPage`，失败留页且显示可理解状态；不调用 `CommissionSubsystem::SetPreparationSlot/ConfirmDispatch`。
- [ ] **Step 4: 两个过滤器 GREEN；复跑 `Dispatch.Widget.CommissionDispatch` 和预览 Coordinator 相关组，逐项记录既有失败与新增失败**。

### Task 3: UE 实际预览与视觉核对

**Files:**
- Modify: `Source/DispatchEditor/Private/Tests/CommissionDeskPanTests.cpp`（样例数据与渲染断言）
- Modify: `docs/narrative/STATUS.md`、`docs/narrative/IMPLEMENTATION_MATRIX.md`、`docs/narrative/handoffs/`（本地交接）
- Optional: `Source/DispatchEditor/Private/StoryWorkspace/StoryCommissionPreviewSession.cpp`（仅当现有隔离预览入口无法传入样例时，加显式测试入口；不得默默覆盖已有预览数组）

**Interfaces:**
- Consumes: Tasks 1–2 的交互页面与预览草稿。
- Produces: 1920×1080 清单/卷宗截图及自动测试日志；不产出正式物品资产。

- [ ] **Step 1: 写渲染/交互 RED 测试**：由隔离预览入口显式注入四项样例及无图标/长名/六项滚动夹具，断言底图、三层阴影和按钮槽未被改写；记录行/详情/按钮几何在纸面内部。
- [ ] **Step 2: 运行 RED；仅在必要时补显式测试入口与样例图标，不修改正式 DA、背包或源 PNG**。
- [ ] **Step 3: 构建并运行 `Dispatch.Widget.CommissionDispatch` 和 Story Workspace 委托预览测试；检查 1920×1080 两视角渲染及实际 PIE 点击。旧 GREEN 先标需要重验，只有本工作区新证据通过才恢复。**
- [ ] **Step 4: 更新本地 STATUS/Matrix/handoff，交付截图、日志、备份路径、自动通过/失败和待用户原生窗口审美验收项；不提交/推送。**
