# Inventory Letter Read Button Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the notebook item-detail button open selected collected letters and return focus to the notebook on close.

**Architecture:** `UPlayerNotebookWidget` derives action visibility from the selected `FDispatchItemInfo` and creates the configured `ULetterWidgetBase`. `ULetterWidgetBase` optionally restores UI focus to a return widget instead of always restoring game input.

**Tech Stack:** Unreal Engine 5.6 C++, UMG, Unreal Automation Tests.

## Global Constraints

- Do not edit, delete, move, or overwrite any `.uasset`; all current asset changes belong to the user.
- Support both the preferred `Button_ReadLetter` name and the current `Button_139` name.
- Ordinary items and empty selection must keep the action button collapsed.
- Reading an inventory letter must not add, remove, or duplicate inventory entries.
- Work directly in the current `main` checkout as previously authorized; do not stage or commit.

---

### Task 1: Inventory Letter Action State

**Files:**
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Modify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`
- Modify: `Source/Dispatch/Private/Tests/InventoryWidgetTests.cpp`

**Interfaces:**
- Consumes: `FDispatchItemInfo::LetterDefinition`.
- Produces: `TSubclassOf<ULetterWidgetBase> LetterWidgetClass`, optional `Button_ReadLetter`/`Text_ReadLetter` bindings, `bool CanReadSelectedLetter() const`.

- [ ] **Step 1: Add failing presentation tests**

Bind a real `UButton` and `UTextBlock` to a notebook, show an ordinary item, a valid letter item, and then clear the preview. Assert collapsed, visible with label `阅读`, and collapsed respectively. Add reflection coverage for `LetterWidgetClass` restricted to `ULetterWidgetBase`.

- [ ] **Step 2: Verify RED**

Build `DispatchEditor`. Expected: compilation fails because the new public contract and bindings do not exist.

- [ ] **Step 3: Implement action state**

Resolve `Button_ReadLetter`, falling back to `Button_139`; resolve `Text_ReadLetter`, falling back to the button's direct `UTextBlock` content. Bind the click once, update visibility in `ShowInventoryItemPreview`, collapse in `ClearInventoryItemPreview`, and remove the delegate in `NativeDestruct`.

- [ ] **Step 4: Verify GREEN**

Run `Dispatch.Widget.Inventory.Preview.LetterAction`. Expected: all action-state assertions pass.

---

### Task 2: Open Reader and Return Focus

**Files:**
- Modify: `Source/Dispatch/Public/Widget/LetterWidgetBase.h`
- Modify: `Source/Dispatch/Private/Widget/LetterWidgetBase.cpp`
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Modify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`
- Modify: `Source/Dispatch/Private/Tests/LetterWidgetTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/InventoryWidgetTests.cpp`

**Interfaces:**
- Produces: `void SetReturnWidget(UUserWidget*)`, `UUserWidget* GetReturnWidget() const`, and `bool OpenSelectedLetter()`.

- [ ] **Step 1: Add failing lifecycle tests**

Assert the letter widget stores an optional return widget, and assert the notebook rejects opening with no selected letter or no configured reader class without mutating inventory.

- [ ] **Step 2: Verify RED**

Build the editor target. Expected: missing return-widget and open-selected-letter APIs.

- [ ] **Step 3: Implement minimal opening flow**

Create `LetterWidgetClass` with the notebook's owning player, initialize it without a mailbox, call `SetReturnWidget(this)`, and add it to the viewport. On letter close, restore UI-only input and cursor focus to a valid return widget; otherwise retain the existing game-input restoration used by mailbox letters.

- [ ] **Step 4: Verify GREEN and regressions**

Run `Dispatch.Widget.Inventory`, `Dispatch.UI.LetterWidget`, `Dispatch.Inventory`, `Dispatch.Items.Mailbox`, `Dispatch.Commission`, and `Dispatch.Story`; all discovered tests must pass with zero failures.

---

### Task 3: Final Verification

- [ ] Build `DispatchEditor` and require exit code 0.
- [ ] Run `git diff --check` and scan new files for trailing whitespace.
- [ ] Confirm only C++/test files were authored by Codex and all user `.uasset` changes remain untouched.
- [ ] Hand off the single Blueprint default: set `WBP_PlayerNotebook.LetterWidgetClass` to `WBP_LetterUI`.
