# Inventory Detail Panel Visibility Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Hide the inventory `DetailPanel` whenever no inventory item is selected, show it after a valid item selection, and reset it when the inventory grid is rebuilt after returning to the page.

**Architecture:** `UPlayerNotebookWidget` remains the sole owner of inventory selection and preview presentation. It resolves the optional Blueprint panel named `DetailPanel`; `ShowInventoryItemPreview` reveals it, while `ClearInventoryItemPreview` collapses it. Existing grid rebuild behavior already clears selection and therefore becomes the page-return reset path.

**Tech Stack:** Unreal Engine 5.6, C++ UMG, Unreal Automation Tests.

## Global Constraints

- Do not edit `WBP_PlayerNotebook.uasset` or any other Blueprint asset.
- Preserve all unrelated dirty worktree changes.
- Treat `DetailPanel` as an optional binding so code-only tests and older Blueprint variants remain safe.

---

### Task 1: Drive DetailPanel visibility from selection state

**Files:**
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Modify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`
- Test: `Source/Dispatch/Private/Tests/InventoryWidgetTests.cpp`

**Interfaces:**
- Consumes: `UPlayerNotebookWidget::ShowInventoryItemPreview(const FDispatchItemInfo&)` and `UPlayerNotebookWidget::ClearInventoryItemPreview()`.
- Produces: optional `UPanelWidget* DetailPanel` binding whose visibility mirrors whether an inventory preview is active.

- [ ] **Step 1: Write the failing test**

Extend `FInventoryPreviewPresentationTest` with a `UPanelWidget` test binding named `DetailPanel`. Assert that `ClearInventoryItemPreview()` collapses it, `ShowInventoryItemPreview()` makes it `HitTestInvisible`, and a later clear collapses it again.

- [ ] **Step 2: Run the focused test to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Widget.Inventory.Preview.Presentation;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: the new visibility assertion fails because production code does not bind or update `DetailPanel`.

- [ ] **Step 3: Implement the minimal visibility behavior**

Add an optional `UPanelWidget` binding named `DetailPanel`, resolve it by name as a compatibility fallback, set it to `ESlateVisibility::HitTestInvisible` in `ShowInventoryItemPreview`, and set it to `ESlateVisibility::Collapsed` in `ClearInventoryItemPreview`.

- [ ] **Step 4: Verify GREEN and regression coverage**

Run the focused preview test, then all `Dispatch.Widget.Inventory` tests. Expected: both runs complete with zero failed tests.

- [ ] **Step 5: Compile the editor target**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: `Build succeeded` with no C++ compile errors.

