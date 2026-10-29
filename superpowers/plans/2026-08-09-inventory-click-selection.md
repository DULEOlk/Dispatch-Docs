# Inventory Click Selection Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make slot selection frames and right-page inventory details respond only to clicks.

**Architecture:** `UInventorySlotWidget` exposes an occupied-slot click delegate and controls `Image_45` visibility. `UPlayerNotebookWidget` maintains one selected slot, transfers selection on click, and presents or clears details during selection and rebuild.

**Tech Stack:** Unreal Engine 5.6, C++, UMG, Unreal Automation Testing

## Global Constraints

- Hover and unhover must not update right-page details.
- Empty-slot clicks must not change selection.
- Blueprint remains limited to widget names, layout, and artwork.

---

### Task 1: Add Click Selection Behavior

**Files:**
- Modify: `Source/Dispatch/Public/Widget/InventorySlotWidget.h`
- Modify: `Source/Dispatch/Private/Widget/InventorySlotWidget.cpp`
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Modify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`
- Test: `Source/Dispatch/Private/Tests/InventoryWidgetTests.cpp`

**Interfaces:**
- Produces: `FOnInventorySlotSelected`, `UInventorySlotWidget::SetSelected(bool)`, and notebook click-selection handling.

- [ ] **Step 1: Add failing tests**

Assert that `Image_45` starts collapsed, occupied clicks broadcast selection, empty clicks do not, click selection populates details, selection transfers between slots, hover does nothing, and rebuild clears selection.

- [ ] **Step 2: Verify tests fail**

Build `DispatchEditor`, then run `Dispatch.Widget.Inventory`. Expected: failure because click selection and selection-frame state do not exist.

- [ ] **Step 3: Implement slot click selection**

Bind `Image_45` and `Button_HitArea::OnClicked`; expose selection state and broadcast only for occupied slots.

- [ ] **Step 4: Implement notebook single selection**

Subscribe generated slots to the click delegate, clear the previous selected slot, select the clicked slot, and populate the right page. Remove hover subscriptions and handlers.

- [ ] **Step 5: Verify**

Build and run `Dispatch.Widget.Inventory`, `Dispatch.Inventory`, and `Dispatch.Widget.QuestNotebook`. Expected: all commands exit successfully.

- [ ] **Step 6: Commit**

Commit only the inventory widget headers, implementations, and tests with message `feat: select inventory items on click`.
