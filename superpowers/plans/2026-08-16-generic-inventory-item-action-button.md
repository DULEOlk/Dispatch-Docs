# Generic Inventory Item Action Button Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rename the notebook's letter-specific action button to a generic item action and route the existing letter view behavior through an extensible action state.

**Architecture:** `UPlayerNotebookWidget` owns one optional `Button_ItemAction`/`Text_ItemAction` pair. It derives a private action state from the selected preview item, updates the shared button, and dispatches clicks to the existing letter-domain method; equip and unequip values are reserved but cannot be produced until a separate equipment system exists.

**Tech Stack:** Unreal Engine 5.6 C++, UMG, Unreal Automation Tests.

## Global Constraints

- Keep the design and implementation plan local under the Git-ignored `docs` directory; do not stage or commit them.
- Do not edit, delete, move, or overwrite any `.uasset`; all current asset changes belong to the user.
- Use only `Button_ItemAction` and `Text_ItemAction`; remove runtime fallback to `Button_ReadLetter`, `Text_ReadLetter`, and `Button_139`.
- Preserve `CanReadSelectedLetter()` and `OpenSelectedLetter()` as letter-specific domain APIs.
- Current behavior supports only `None` and `View`; do not implement equipment storage or effects in this change.
- Work directly in the current checkout and do not stage or commit source changes.

---

### Task 1: Generic Action Presentation Contract

**Files:**
- Modify: `Source/Dispatch/Private/Tests/InventoryWidgetTests.cpp`
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Modify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`

**Interfaces:**
- Consumes: `FDispatchItemInfo::LetterDefinition` and the existing `bool OpenSelectedLetter()`.
- Produces: optional `Button_ItemAction`/`Text_ItemAction` bindings, `EInventoryItemAction`, `ResolveInventoryItemAction()`, `BindInventoryItemAction()`, `RefreshInventoryItemAction()`, and `HandleItemActionClicked()`.

- [ ] **Step 1: Change the presentation test to require the new UMG names**

Construct the widgets with the official names:

```cpp
UButton* ActionButton = Notebook->WidgetTree->ConstructWidget<UButton>(
	UButton::StaticClass(),
	TEXT("Button_ItemAction"));
UTextBlock* ActionLabel = Notebook->WidgetTree->ConstructWidget<UTextBlock>(
	UTextBlock::StaticClass(),
	TEXT("Text_ItemAction"));
ActionButton->SetContent(ActionLabel);
```

Keep the ordinary item, letter, and clear-selection assertions, but require the letter label to be `查看`.

- [ ] **Step 2: Run the focused test to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' `
  'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' `
  '-ExecCmds=Automation RunTests Dispatch.Widget.Inventory.Preview.LetterAction;Quit' `
  -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: the test fails because the existing implementation searches only for `Button_ReadLetter`/`Button_139` and never updates `Button_ItemAction`.

- [ ] **Step 3: Add the generic action state and bindings**

Declare the non-reflected state:

```cpp
enum class EInventoryItemAction : uint8
{
	None,
	View,
	Equip,
	Unequip
};
```

Rename the optional widget properties to:

```cpp
TObjectPtr<UButton> Button_ItemAction;
TObjectPtr<UTextBlock> Text_ItemAction;
```

Replace the letter-specific binding helpers with:

```cpp
EInventoryItemAction ResolveInventoryItemAction() const;
void BindInventoryItemAction();
void RefreshInventoryItemAction();

UFUNCTION()
void HandleItemActionClicked();
```

Store `CurrentInventoryItemAction`, resolve only valid selected letters to `View`, set the label to `查看`, collapse `None`, and route `View` clicks to `OpenSelectedLetter()`. Include labels for `Equip` and `Unequip`, but do not generate or execute those states yet.

- [ ] **Step 4: Build and rerun the focused test for GREEN**

Build:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' `
  DispatchEditor Win64 Development `
  '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' `
  -WaitMutex -FromMsBuild
```

Then rerun `Dispatch.Widget.Inventory.Preview.LetterAction`. Expected: build exits 0 and the focused test reports zero errors.

---

### Task 2: Regression and Naming Verification

**Files:**
- Verify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Verify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`
- Verify: `Source/Dispatch/Private/Tests/InventoryWidgetTests.cpp`

**Interfaces:**
- Consumes: the generic action contract from Task 1.
- Produces: verified C++ source ready for the user to rename the matching UMG controls.

- [ ] **Step 1: Run the inventory widget regression group**

Run `Automation RunTests Dispatch.Widget.Inventory;Quit` with the same unattended editor command. Expected: all inventory widget tests pass with zero errors.

- [ ] **Step 2: Verify old runtime names are absent**

Run:

```powershell
rg -n "Button_ReadLetter|Text_ReadLetter|Button_139|HandleReadLetterClicked|RefreshInventoryLetterAction|BindInventoryLetterAction" Source/Dispatch
```

Expected: no matches.

- [ ] **Step 3: Verify diff quality and preserve assets**

Run `git diff --check` and inspect `git diff` limited to the three files. Expected: no whitespace errors and no Codex-authored `.uasset` changes.

- [ ] **Step 4: Hand off the required Blueprint rename**

Tell the user to rename the existing `WBP_PlayerNotebook` button to `Button_ItemAction`, rename its child text to `Text_ItemAction`, enable `Is Variable`, compile, and save. Decorative children over the button must be `Hit Test Invisible`.
