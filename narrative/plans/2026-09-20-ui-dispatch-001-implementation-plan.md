# UI-DISPATCH-001 委托选择到派发确认 Implementation Plan

> **执行约束（适用于任意 AI/开发者）：** 在当前本地 checkout 中逐项执行本计划，不创建 worktree。每批生产改动都遵循 RED → GREEN → 聚焦回归 → 记录更新；开始修改业务代码、Blueprint、Config 或 `.uasset` 前，必须先向用户汇报并取得明确授权。

**Goal:** 在 Story Workspace 的隔离预览中完成一条顺畅、可重复测试的“选择委托卡→打开派遣卷宗→配置交付物草稿→返回或确认派发”UI 流程。

**Architecture:** 保留 `UCommissionSelectionUIBase` 作为委托选择页，保留 `UCommissionDispatchUIBase` 作为卷宗界面，并为卷宗增加仅属于 UI 的交付物草稿和返回/确认事件。`FStoryUIPreviewCoordinator` 负责两个页面之间的切换，`UStoryCommissionPreviewSession` 提供正式 Commission/NPC DA 的临时副本及显式配置的预览交付物；确认只生成本次隔离预览结果，不写正式资产、存档或真实 `UCommissionSubsystem` 实例。以后地图接线再把同一界面事件适配到真实派遣事务。

**Tech Stack:** Unreal Engine 5.6、C++20/UHT、UMG Widget Blueprint、Dispatch/DispatchEditor 模块、Unreal Automation Tests、Story Workspace 隔离 PIE 预览。

**Spec:** `docs/narrative/ui/COMMISSION_NOTICE_UI.md` 的“派遣入口”和“派遣卷宗”章节；阶段顺序见 `docs/narrative/GAMEPLAY_FLOW.md#10-迭代验证而非提前设计完整七天`；父级要求为 `DEMO-COM-001`、`DEMO-AUTH-001`、`DEMO-VIS-001`。

## Global Constraints

- 直接使用 `F:\Epic Games\Unreal Projects\Dispatch`，不创建 worktree。
- 开工前重新读取根 `AGENTS.md`、叙事入口、Matrix、Status、本计划和最新 handoff。
- 保留当前所有 tracked/untracked 改动；尤其不能覆盖已经修改的 `WBP_CommissionSelection.uasset`。
- 当前 NPC 由预览配置的 Commission/NPC 对提供；本任务不增加 NPC 选择页。
- 当前只完成 UI 层的派发确认结果；不生成真实跨日委托实例，不消耗真实物品，不占用真实 NPC，不写 SaveGame。
- 不包含公告板接取、地图入口、NPC 动作/对话、教程、Day 1 编排、跨日返回、报告和撰写。
- 不改变最终视觉风格；只允许为明确交互状态补必要的文字、显隐、禁用和反馈。
- “暂不交付”语义仍未确认；本任务不得擅自把它解释为空准备派遣、返回或暂缓。返回使用明确的返回入口。
- 不从 Commission 分支隐藏条件反推预览交付物，避免在派遣前泄露正确答案。预览交付物必须由 Story Workspace 显式配置。
- 每完成一个可验证批次同时更新 `STATUS.md`、`IMPLEMENTATION_MATRIX.md` 和 `handoffs/2026-09-20-ui-dispatch-001.md`。
- 不提交、不推送；只有用户另行授权后才能执行 Git 提交。

## Review Focus

- 委托数量为 0、超过 3 或含空引用时：给出明确错误，不打开空白/越界界面。
- 选择委托与 NPC 配对错位时：卷宗必须显示被点击委托对应的那一行 NPC，不能沿用上一张卡的数据。
- 重复点击卡片或确认按钮时：一次操作只能触发一次页面切换或一次预览确认。
- 返回后重新选择时：上一份委托的交付物草稿、错误和确认状态必须全部清除。
- 正式 Commission/NPC DA、当前 GameInstance、存档和真实 Commission 实例：预览前后均不得被修改或变脏。

---

### Task 1: 锁定现有资产基线并为短流程补 RED 测试

**Files:**
- Modify: `Source/Dispatch/Private/Tests/CommissionDispatchPresentationTests.cpp`
- Modify: `Source/DispatchEditor/Private/Tests/StoryUIPreviewCoordinatorTests.cpp`
- Inspect only: `Content/BluePrints/Widget/Menu/Quest/WBP_CommissionSelection.uasset`
- Inspect only: `Content/BluePrints/Widget/Menu/Quest/WBP_CommissionCard.uasset`
- Inspect only: `Content/BluePrints/Widget/Menu/Quest/WBP_CommissionDispatch.uasset`
- Update: `docs/narrative/STATUS.md`
- Update: `docs/narrative/IMPLEMENTATION_MATRIX.md`
- Update: `docs/narrative/handoffs/2026-09-20-ui-dispatch-001.md`

**Interfaces:**
- Consumes: existing `UCommissionSelectionUIBase::SelectCommissionByIndex(int32)` and `UStoryCommissionPreviewSession::Present(...)`.
- Produces: executable RED expectations for dossier draft state, return, reconnection and single confirmation.

- [ ] **Step 1: Re-read workspace state and protect existing asset changes**

Run `git status --short --untracked-files=all`, record the exact baseline in the handoff, and inspect the three Widget Blueprint parents/bindings in Unreal Editor without saving. Record whether `WBP_CommissionSelection` contains user changes not represented by C++ tests. Do not reparent or compile/save any asset in this step.

- [ ] **Step 2: Reproduce the focused baseline**

Run these filters separately so Unreal does not treat later filters as unknown console commands:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' `
  'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' `
  '-ExecCmds=Automation RunTests Dispatch.Widget.CommissionSelection;Quit' `
  -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache

& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' `
  'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' `
  '-ExecCmds=Automation RunTests Dispatch.Widget.CommissionDispatch;Quit' `
  -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache

& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' `
  'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' `
  '-ExecCmds=Automation RunTests Dispatch.Editor.StoryWorkspace.PreviewCoordinator.CommissionLifecycle;Quit' `
  -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected baseline: existing selection/card behavior is reported exactly, including the previously recorded idle-dim mismatch if it still reproduces; CommissionDispatch presentation and current selection→dossier preview should pass. Do not fix unrelated animation behavior in this step.

- [ ] **Step 3: Add the missing Widget interaction RED**

Add `Dispatch.Widget.CommissionDispatch.InteractionState` to require these public contracts, which do not yet exist:

```cpp
DECLARE_MULTICAST_DELEGATE(FOnCommissionDispatchReturnRequestedNative);
DECLARE_MULTICAST_DELEGATE_OneParam(
    FOnCommissionDispatchPreviewConfirmedNative,
    const FCommissionDispatchPreviewResult&);

bool InitializePreviewDraft(
    const UCommissionDefinition* Commission,
    const UNPCDefinition* NPC,
    const TArray<FCommissionPreparationViewData>& AvailablePreparations);
bool SelectPreparationSlot(int32 SlotIndex);
bool ChoosePreviewPreparation(int32 PreparationIndex);
bool RequestPreviewDispatch();
void RequestReturn();
```

The test must pin: exactly three visible draft slots for the current prototype, selecting and replacing a slot, invalid indices leaving state unchanged, one native return event, and one confirmation event even if the confirm method is called twice.

- [ ] **Step 4: Add the missing coordinator lifecycle RED**

Extend `Dispatch.Editor.StoryWorkspace.PreviewCoordinator.CommissionLifecycle` so it requires:

```text
选择第二张卡
→ 第二行 Commission/NPC 出现在卷宗
→ 选择一个显式预览交付物
→ 返回
→ 原选择页重新进入 viewport 且旧草稿清空
→ 重新选择第一张卡
→ 确认只产生一次 preview result
```

Also assert that the source asset package remains clean and `UCommissionSubsystem::GetCommissionInstances()` remains empty.

- [ ] **Step 5: Build and verify correct RED**

Run the DispatchEditor build. Expected: compile or focused tests fail only because the new preview-draft types/API and coordinator handlers are absent. Record the first relevant failure in Status/Matrix/handoff before production implementation.

### Task 2: 为派遣卷宗增加最小 UI 草稿状态

**Files:**
- Modify: `Source/Dispatch/Public/Widget/CommissionDispatchUIBase.h`
- Modify: `Source/Dispatch/Private/Widget/CommissionDispatchUIBase.cpp`
- Modify: `Source/Dispatch/Private/Tests/CommissionDispatchPresentationTests.cpp`

**Interfaces:**
- Consumes: `UCommissionDefinition::MaxPreparationSlots` and existing `FCommissionPreparationViewData`.
- Produces: `FCommissionDispatchPreviewResult`, preview-draft methods, return/confirm delegates, and Blueprint-readable slot/option/error state.

- [ ] **Step 1: Add the smallest serializable preview result**

Implement a UI-only result containing no outcome or save/runtime state:

```cpp
USTRUCT(BlueprintType)
struct FCommissionDispatchPreviewResult
{
    GENERATED_BODY()
    UPROPERTY(BlueprintReadOnly) FName CommissionID = NAME_None;
    UPROPERTY(BlueprintReadOnly) FName NPCID = NAME_None;
    UPROPERTY(BlueprintReadOnly) TArray<FCommissionPreparationRef> Preparations;
};
```

- [ ] **Step 2: Add preview draft state without Commission business rules**

Store the selected Commission/NPC IDs, three `FCommissionPreparationRef` slots, the explicit preview options, selected slot index, visible error text, and `bPreviewDispatchConfirmed`. `InitializePreviewDraft` must reject a null Commission/NPC, reject a definition whose `MaxPreparationSlots` is not 3 for this prototype, clear all previous state, then call the existing presentation mapping.

- [ ] **Step 3: Implement slot selection and replacement**

`SelectPreparationSlot` accepts only indices 0–2. `ChoosePreviewPreparation` requires a selected slot and valid configured option; it replaces the selected slot and rejects the same preparation appearing in two different slots. It does not evaluate hidden Commission branches and does not claim the choice is correct.

- [ ] **Step 4: Implement return and idempotent preview confirmation**

`RequestReturn` only accepts an active, unconfirmed draft: the first call clears/deactivates local state and broadcasts once; repeated calls are ignored. `RequestPreviewDispatch` rejects missing Commission/NPC state, constructs the UI-only result, marks the draft confirmed, broadcasts once, disables subsequent return/confirmation, and never calls `UCommissionSubsystem`.

- [ ] **Step 5: Refresh optional bound controls**

Add optional bindings using the existing functional Widget naming convention:

```text
List_AvailablePreparations
Button_PreparationSlot0 / 1 / 2
Text_PreparationSlot0 / 1 / 2
Button_Dispatch
Button_Return
Text_Error
```

Missing optional controls must not crash the pure presentation test. Slot buttons call `SelectPreparationSlot`, the available-preparation list item click calls `ChoosePreviewPreparation`, and the action buttons call only the new UI-draft methods.

- [ ] **Step 6: Return the Widget tests to GREEN**

Run `Dispatch.Widget.CommissionDispatch`. Expected: presentation, authored binding and interaction state tests all pass; no hidden OutcomeID is exposed and no formal asset becomes dirty.

### Task 3: 让 Story Workspace 预览驱动完整短流程

**Files:**
- Modify: `Source/DispatchEditor/Public/StoryWorkspace/StoryCommissionPreviewSession.h`
- Modify: `Source/DispatchEditor/Private/StoryWorkspace/StoryCommissionPreviewSession.cpp`
- Modify: `Source/DispatchEditor/Public/StoryWorkspace/StoryUIPreviewRequest.h`
- Modify: `Source/DispatchEditor/Public/StoryWorkspace/StoryUIPreviewCoordinator.h`
- Modify: `Source/DispatchEditor/Private/StoryWorkspace/StoryUIPreviewCoordinator.cpp`
- Modify: `Source/DispatchEditor/Private/Tests/StoryCommissionAssetPreviewTests.cpp`
- Modify: `Source/DispatchEditor/Private/Tests/StoryUIPreviewCoordinatorTests.cpp`

**Interfaces:**
- Consumes: formal Commission/NPC DA pairs and explicit editor-only preview preparations.
- Produces: stable selected-pair lookup, selection↔dossier navigation, and one isolated preview confirmation result.

- [ ] **Step 1: Add explicit preview preparation configuration**

Add this field to `FStoryCommissionPreviewAssetEntry`:

```cpp
UPROPERTY(EditAnywhere, meta=(DisplayName="预览交付物"))
TArray<FCommissionPreparationViewData> PreviewPreparations;
```

Validation rejects invalid references, duplicate `(Type, ContentID)` pairs, non-positive `AvailableCount`, and more than a small preview list of 12 options. It never reads Commission branch requirements to generate these options.

- [ ] **Step 2: Make session lookup return the exact selected pair**

Replace display-only lookup with an exact method:

```cpp
bool InitializeDraft(
    UCommissionDefinition* Selected,
    UCommissionDispatchUIBase* Widget) const;
```

It finds the snapshot whose Commission pointer is exactly `Selected`, then passes that snapshot's Commission, NPC and configured preview preparations to `InitializePreviewDraft`. A foreign or expired selection returns false.

- [ ] **Step 3: Bind dossier return and confirmation events**

After card selection, the coordinator removes the selection page, creates the dossier, binds the two native events, initializes the exact selected pair, then transfers UI focus. On return, it removes the dossier, clears its delegates/state, restores the existing selection page, clears selected index and returns focus. On confirmation, it stores one `FCommissionDispatchPreviewResult` for inspection and prevents duplicate processing.

- [ ] **Step 4: Make StopPreview clear every owned object**

Reset must remove both pages, unbind all delegates, clear the preview result, release snapshots and restore the coordinator to Stopped without writing GameInstance, SaveGame or source packages.

- [ ] **Step 5: Return session/coordinator tests to GREEN**

Run these separately:

```text
Dispatch.Editor.StoryWorkspace.Commission
Dispatch.Editor.StoryWorkspace.PreviewCoordinator.CommissionLifecycle
Dispatch.Editor.StoryWorkspace.PreviewCoordinator.CommissionAssets
```

Expected: selection, exact data mapping, return/reselect, single confirmation, restart refresh and cleanup all pass.

### Task 4: 两页 Widget 交互，分四次独立验收

用户 2026-09-21 的两张截图澄清：卷宗中的三格只有 Image/Text 装饰；新建的 `Content/BluePrints/Widget/Menu/Quest/NewWidgetBlueprint.uasset` 目前也只有纸张背景，没有交付物列表。不得把它们视为已经可点击或已完成的列表。本 Task 按 4A→4B→4C→4D 顺序执行；每一段完成后停下，更新 Matrix/Status/handoff 并由用户实际点击确认，再进入下一段。`NewWidgetBlueprint.uasset` 是用户未跟踪资产，改动前先核对当前内容并保护；不得覆盖或删除。

#### Task 4A：点击三格并打开/关闭交付物页

**Files:** `Content/BluePrints/Widget/Menu/Quest/WBP_CommissionDispatch.uasset`；`Content/BluePrints/Widget/Menu/Quest/NewWidgetBlueprint.uasset`（仅在核对后、获授权时修改）；必要时最小修改 `Source/Dispatch/Public/Widget/CommissionDispatchUIBase.h`、`Source/Dispatch/Private/Widget/CommissionDispatchUIBase.cpp` 和 `Source/Dispatch/Private/Tests/CommissionDispatchPresentationTests.cpp`。`WBP_CommissionSelection`、`WBP_CommissionCard` 只读。

**Contract:** 入口是当前卷宗的槽位 0/1/2；已有 `SelectPreparationSlot(int32)` 保存所选槽。交付物页此时只承担打开/关闭，不展示列表、不写槽、不确认派遣。界面可以用明确的临时“返回卷宗”按钮关闭；不得把“暂不交付”用作返回。

- [ ] 记录完整 `git status --short --untracked-files=all`；只读打开两个 Widget，确认三格的 Image/Text 层级、是否已有按钮/覆盖层、`NewWidgetBlueprint` 的父类/根控件/现有用户编辑，并记录截图或层级证据。
- [ ] 先增加或扩展一个仅验证“槽 0/1/2 → 记录正确索引 → 交付页显示 → 关闭后回到同一卷宗”的自动化用例；取得预期 RED。若无法在命令行验证实际 UMG 点击，至少验证公开打开/关闭合同，并把真实点击列为必需人工验收。
- [ ] 以透明按钮覆盖三个原图片区域，保留原图、加号、文字和位置；三个按钮分别调用 `SelectPreparationSlot(0/1/2)`，成功后打开同一个交付物页面，不新建三个页面。关闭仅移除交付物页面并恢复卷宗输入，不清除已选槽。
- [ ] 为新页面提供清楚可见的关闭/返回入口；重复点击槽位不得叠加多个页面，打开失败不得留下遮挡输入的透明层。
- [ ] 只编译/保存实际触碰的 Widget；检查 `git status` 是否出现无关资产变化；重跑 `Dispatch.Widget.CommissionDispatch` 与相关 Story Workspace 预览用例。
- [ ] 用户在 Story Workspace 中分别点击左、中、右三格，确认都打开同一交付页、关闭后仍在原卷宗、没有列表也没有放入交付物。此项通过才把 4A 标为完成；不能据此宣称 Task 4 完成。

#### Task 4B：交付物页显示可选列表

**Files:** `NewWidgetBlueprint.uasset`；必要时在 `Source/Dispatch` 新增独立的交付物条目 Widget/ViewModel 并测试；已有 `FCommissionPreparationViewData` 和 `UCommissionPreparationViewModel` 可作为数据合同，先评估复用，避免第二份交付物数据源。

**Contract:** 页只读取 Story Workspace 显式配置的 `PreviewPreparations`；显示名称、类型和可用数量（有图标时显示图标）；无项目时显示“暂无可交付物”。不得从 Commission 隐藏结果分支反推正确选项。此段列表只读，点击条目不回填。

- [ ] 为 0 项、2 项、超过可见高度的列表写数据/展示测试并取得 RED。
- [ ] 在交付页加入滚动列表和条目 Widget，按 `GetAvailablePreviewPreparations()` 的顺序展示；条目数据与正式 DA 临时快照保持隔离。
- [ ] 编译/保存触碰资产，运行聚焦测试；用户验证列表可读、可滚动、空态清楚、返回仍正常。通过后才进入 4C。

#### Task 4C：点击条目并回填所选槽

**Files:** `NewWidgetBlueprint.uasset`、`WBP_CommissionDispatch.uasset`；必要时补 `CommissionDispatchUIBase` 的条目事件/刷新代码与 Widget 自动化测试。

**Contract:** 点击条目对应 `ChoosePreviewPreparation(PreparationIndex)`；成功后关闭交付页，卷宗仅更新此前选择的槽位；可再次打开并替换。选择无效项、重复占用其他槽位时显示错误且保留原槽内容，不静默失败。

- [ ] 写并运行“选第 2 格→点击条目→只填第 2 格；替换；重复项拒绝”RED。
- [ ] 接通列表点击、槽位文字/图标刷新和失败反馈；不得以列表视觉选中代替草稿实际更新。
- [ ] 运行聚焦测试，用户点击验证三格都能填入/替换，错误时原值不丢；通过后才进入 4D。

#### Task 4D：补齐后续操作与完整 Widget 回归

**Files:** `WBP_CommissionDispatch.uasset`、`NewWidgetBlueprint.uasset`；仅在证明需要时修改 `WBP_CommissionSelection.uasset`/`WBP_CommissionCard.uasset` 与对应测试。保护 Selection 的既有用户修改。

- [ ] 定义并实现明确的槽位清空入口、选中/已填/不可用反馈、独立的卷宗返回入口与预览派遣确认按钮；“暂不交付”继续保持未决，不挪作其他动作。
- [ ] 将返回、重选、单次确认与 StopPreview 清理接到真实 Widget；用户点击验证不会叠页、串槽、重复确认或卡住输入。
- [ ] 复跑 `Dispatch.Widget.CommissionSelection` 和 `Dispatch.Widget.CommissionDispatch`，如仍有既有 `HoverDimsPeers`/`IdleDimOption` 失败，单独记录原因与是否阻塞，不得悄悄改写用户视觉设置。编译并仅保存触碰资产；无关 Content 变化立即停止。
- [ ] Task 4 完成须 4A～4D 各自有自动化结果与人工点击结果；之后才进入 Task 5 的整条短流程验收。

### Task 5: 人工验收、完整回归和交接

**Files:**
- Update: `docs/narrative/IMPLEMENTATION_MATRIX.md`
- Update: `docs/narrative/STATUS.md`
- Update: `docs/narrative/handoffs/2026-09-20-ui-dispatch-001.md`

**Interfaces:**
- Produces: current-workspace build/test evidence and one user-observable acceptance result.

- [ ] **Step 1: Run a fresh editor build**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' `
  DispatchEditor Win64 Development `
  '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' `
  -WaitMutex -FromMsBuild
```

Expected: exit 0 with no new UHT or linker error.

- [ ] **Step 2: Run focused and adjacent regressions separately**

Run these filters one process at a time and record discovered/passed/failed counts:

```text
Dispatch.Widget.CommissionSelection
Dispatch.Widget.CommissionDispatch
Dispatch.Editor.StoryWorkspace.Commission
Dispatch.Editor.StoryWorkspace.PreviewCoordinator.CommissionLifecycle
Dispatch.Editor.StoryWorkspace.PreviewCoordinator.CommissionAssets
Dispatch.Commission
```

Any previously GREEN matrix item affected by a failure is marked “需要重验” before further changes.

- [ ] **Step 3: Perform the manual Story Workspace acceptance**

Using at least two formal Commission/NPC pairs and at least two explicit preview preparations:

```text
启动委托派发预览
→ 选择第二张委托卡
→ 核对卷宗显示第二行委托/NPC
→ 选择交付槽并放入、替换交付物
→ 返回选择页
→ 重新选择第一张卡，确认旧草稿为空
→ 点击确认派发
→ 看见一次明确的“预览确认”反馈
→ 停止并再次启动，确认状态全新且正式 DA 未变脏
```

The user records pass/fail separately for data correctness, click flow, feedback clarity and absence of stuck input.

- [ ] **Step 4: Verify scope and update records**

Run `git diff --check -- Source Config`, inspect tracked/untracked Content changes, and list every touched `.uasset`. Update Matrix/Status/handoff with design, code, automation and manual acceptance as four separate statuses. Do not mark `DEMO-COM-001` or the complete UI chain finished solely because this short preview passes.

## Exit Conditions

`UI-DISPATCH-001` can close only when all of the following are true:

- Story Workspace can open at least two formal Commission/NPC choices.
- Clicking a card always opens the matching dossier.
- Three current prototype slots can receive and replace explicit preview preparations.
- Return restores the selection page and clears the previous draft.
- Confirm emits exactly one isolated preview result and provides visible feedback.
- Stopping/restarting preview clears all transient state and leaves source DA/runtime state unchanged.
- Fresh build and all focused/adjacent filters are GREEN in the current workspace.
- The user completes the manual acceptance and records the result.

## Stop Conditions

Stop implementation, update Status/Matrix/handoff, and request user direction if any of these occurs:

- Completing the flow requires overwriting unexplained existing changes in `WBP_CommissionSelection` or another user asset.
- The current Blueprint hierarchy cannot bind the required controls without a visual/layout redesign.
- A real `UCommissionSubsystem` transaction, StoryDirector, DayFlow, map actor or SaveGame becomes necessary for this UI-only stage.
- The task would need to decide the unresolved meaning of “暂不交付”.
- An unrelated Content asset is saved or a previously GREEN requirement regresses.

## New-Conversation Resume Checklist

The next conversation must:

1. Read root `AGENTS.md`, `docs/narrative/README.md`, `CREATIVE_OVERVIEW.md`, `DEMO_DESIGN.md`, `IMPLEMENTATION_MATRIX.md`, `STATUS.md`, `RULES.md`, `GAMEPLAY_FLOW.md`, `FOUNDATION_DATA_DESIGN.md`, `SYSTEM_DESIGN.md`, `DECISIONS.md`.
2. Read `docs/narrative/ui/COMMISSION_NOTICE_UI.md`, this plan, and `docs/narrative/handoffs/2026-09-20-ui-dispatch-001.md`.
3. Run `git status --short --untracked-files=all` and preserve every existing change.
4. Report the current task step, existing evidence, known failures, exact next RED test, and files that would be touched.
5. Wait for explicit user confirmation before modifying business code, Blueprint, Config or `.uasset`.

## External-AI Handoff Notes

- 如果接手者运行在同一台机器并直接使用 `F:\Epic Games\Unreal Projects\Dispatch`，应以根 `AGENTS.md`、本计划、Matrix、Status 和 handoff 为准；不需要依赖当前聊天记录。
- `docs/` 被 `.gitignore` 忽略，因此这些交接资料只存在于当前本地目录。若接手者使用另一份 clone、云端工作区或不同电脑，必须先把本计划、handoff 及相关叙事文档一并复制过去，否则不能视为无缝接手。
- 接手者不得把已有 dirty files 当作本任务产生的改动，也不得清理、回滚或覆盖它们；尤其先保护当前已修改的 `WBP_CommissionSelection.uasset` 和 DATA-FOUNDATION 源码。
- 本任务的两个主界面是 `WBP_CommissionSelection`（选择委托）与 `WBP_CommissionDispatch`（派遣卷宗）。`WBP_CommissionCard` 只是选择页使用的卡片子控件，仅在测试证明有必要时修改；Story Workspace/C++ 是预览驱动和测试支撑，不是第三个玩家流程页面。
