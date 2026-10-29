# Inventory Grid Size Adjustment Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Enlarge the code-generated 4 x 4 notebook inventory grid so it uses more of the existing scroll area.

**Architecture:** Preserve the current `USizeBox`-wrapped `UInventorySlotWidget` generation and change only its code-owned layout constants. Protect the exact generated geometry through the existing Unreal automation test.

**Tech Stack:** Unreal Engine 5.6, C++, UMG, Unreal Automation Testing

## Global Constraints

- Keep exactly 16 slots arranged in four columns by four rows.
- Use slot size 120 x 105 pixels and 5 pixels of padding on every side.
- Use an explicit WrapBox width of 520 pixels.
- Do not modify Blueprint assets or inventory interaction behavior.

---

### Task 1: Enlarge the Inventory Grid Geometry

**Files:**
- Modify: `Source/Dispatch/Private/Tests/InventoryWidgetTests.cpp`
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`

**Interfaces:**
- Consumes: `UPlayerNotebookWidget::BuildInventoryGrid(const TArray<FDispatchItemInfo>&)`
- Produces: the existing generated 4 x 4 grid with updated constant geometry

- [ ] **Step 1: Update the test with the approved geometry**

Change the grid assertions to these hand-calculated literals:

```cpp
TestEqual(TEXT("Grid wrap width fits four padded slots"), Grid->GetWrapSize(), 520.0f);
TestEqual(TEXT("Generated slot width is fixed"), FirstContainer->GetWidthOverride(), 120.0f);
TestEqual(TEXT("Generated slot height is fixed"), FirstContainer->GetHeightOverride(), 105.0f);
TestEqual(TEXT("Generated slot uses five-pixel padding"), WrapSlot->GetPadding(), FMargin(5.0f));
```

- [ ] **Step 2: Run the focused test and verify the old dimensions fail**

Run `Dispatch.Widget.Inventory.Grid.Build` through `UnrealEditor-Cmd.exe`.

Expected: FAIL because the implementation still returns wrap width 476, slot size 107 x 97, and padding 6.

- [ ] **Step 3: Update the code-owned layout constants**

Set the existing constants in `UPlayerNotebookWidget` to:

```cpp
static constexpr float InventorySlotWidth = 120.0f;
static constexpr float InventorySlotHeight = 105.0f;
static constexpr float InventorySlotPadding = 5.0f;
```

Keep `InventoryGridWrapWidth` derived as four times slot width plus both horizontal padding sides, producing 520 pixels.

- [ ] **Step 4: Build and verify the focused test passes**

Build `DispatchEditor Win64 Development`, then run `Dispatch.Widget.Inventory.Grid.Build`.

Expected: the build and focused test both exit successfully.

- [ ] **Step 5: Run regression suites**

Run `Dispatch.Widget.Inventory`, `Dispatch.Inventory`, and `Dispatch.Widget.QuestNotebook`.

Expected: every suite exits successfully with no failed automation tests.

- [ ] **Step 6: Commit only the two source files**

```powershell
git add -- Source/Dispatch/Private/Tests/InventoryWidgetTests.cpp Source/Dispatch/Public/Widget/PlayerNotebookWidget.h
git commit -m "fix: enlarge notebook inventory grid"
```
