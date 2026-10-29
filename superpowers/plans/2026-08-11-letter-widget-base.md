# Letter Widget C++ Base Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make `WBP_LetterUI` a layout-only Blueprint whose data display, close button, mailbox collection, and input-mode behavior are owned by C++.

**Architecture:** Add a focused `ULetterWidgetBase` that binds the three existing named widgets and owns the letter presentation lifecycle. Extend `AMailbox` with a configurable widget class and create the widget directly during interaction, while retaining `OnLetterOpened` as a compatibility fallback.

**Tech Stack:** Unreal Engine 5.6 C++, UMG, UObject reflection, Unreal Automation Tests.

## Global Constraints

- Do not edit or overwrite any `.uasset`; preserve the user's current Blueprint, StateTree, and Data Asset changes.
- Keep `OnLetterTaken`, `OnLetterOpened`, and `DeliverLetterByID` compatible with existing assets.
- Bind `Text_Name`, `Text_Description`, and `Button_Close` optionally so incomplete Blueprint migration does not fail C++ compilation.
- Work directly in the current `main` checkout as authorized; do not stage or commit files.

---

### Task 1: Letter Widget Base

**Files:**
- Create: `Source/Dispatch/Public/Widget/LetterWidgetBase.h`
- Create: `Source/Dispatch/Private/Widget/LetterWidgetBase.cpp`
- Create: `Source/Dispatch/Private/Tests/LetterWidgetTests.cpp`

**Interfaces:**
- Consumes: `ULetterDefinition::IsDefinitionValid()`, `AMailbox::CompleteLetterReading(APlayerCharacter*)`.
- Produces: `bool InitializeLetter(ULetterDefinition*, AMailbox*, APlayerCharacter*)`, `bool CloseLetter()`, `static FText BuildLetterBody(const ULetterDefinition*)`.

- [ ] **Step 1: Write failing automation tests**

Add tests that require the new class, verify empty/invalid initialization is rejected, verify body segments join with a blank line and fall back to `InventorySummary`, and verify mailbox-backed close collects exactly once.

```cpp
TestEqual(TEXT("Body joins readable segments"),
    ULetterWidgetBase::BuildLetterBody(Letter).ToString(),
    FString(TEXT("First\n\nSecond")));
TestTrue(TEXT("Mailbox letter initializes"),
    Widget->InitializeLetter(Letter, Mailbox, Player));
TestTrue(TEXT("Closing completes mailbox reading"), Widget->CloseLetter());
TestEqual(TEXT("Letter is collected"), Player->FindCollectedLetter(Letter->LetterID), Letter);
```

- [ ] **Step 2: Run the focused test to verify RED**

Run the editor build. Expected: compilation fails because `Widget/LetterWidgetBase.h` and `ULetterWidgetBase` do not exist.

- [ ] **Step 3: Implement the C++ base widget**

Create a Blueprintable `UUserWidget` parent with transient `CurrentLetter`, `SourceMailbox`, and `Reader` references; optional `BindWidget` members for the existing named controls; button binding in `NativeOnInitialized`; UI input activation in `NativeConstruct`; and cleanup in `NativeDestruct`.

`InitializeLetter` validates the asset and mailbox reader, stores context, refreshes title/body, and marks mailbox completion as required. `CloseLetter` refuses to remove the UI if required mailbox completion fails, otherwise restores game input and removes the widget.

- [ ] **Step 4: Build and run focused tests to verify GREEN**

Run `Dispatch.UI.LetterWidget`. Expected: all widget formatting, validation, and close lifecycle tests pass.

---

### Task 2: Mailbox Automatic Widget Presentation

**Files:**
- Modify: `Source/Dispatch/Public/Items/Mailbox.h`
- Modify: `Source/Dispatch/Private/Items/Mailbox.cpp`
- Modify: `Source/Dispatch/Private/Tests/MailboxTests.cpp`

**Interfaces:**
- Consumes: `ULetterWidgetBase::InitializeLetter(ULetterDefinition*, AMailbox*, APlayerCharacter*)`.
- Produces: editable `TSubclassOf<ULetterWidgetBase> LetterWidgetClass` and automatic presentation from `AMailbox::Interact`.

- [ ] **Step 1: Add failing mailbox reflection assertions**

Require `LetterWidgetClass` to exist, be a `ULetterWidgetBase` subclass property, and keep `OnLetterOpened` as a Blueprint event fallback.

```cpp
const FClassProperty* WidgetClassProperty = FindFProperty<FClassProperty>(
    AMailbox::StaticClass(), TEXT("LetterWidgetClass"));
TestEqual(TEXT("Mailbox restricts the UI parent"),
    WidgetClassProperty->MetaClass.Get(), ULetterWidgetBase::StaticClass());
```

- [ ] **Step 2: Build to verify RED**

Expected: the new reflection assertion fails to compile or reports the missing property.

- [ ] **Step 3: Implement automatic mailbox presentation**

Add `LetterWidgetClass` and a transient active widget reference. For a directly delivered letter, resolve the interacting `APlayerCharacter` and its controller, create the configured widget, call `InitializeLetter`, add it to the viewport, and return. If configuration, controller resolution, creation, or initialization fails, invoke the existing `OnLetterOpened` event.

- [ ] **Step 4: Run mailbox and regression suites**

Run `Dispatch.Items.Mailbox`, `Dispatch.Inventory`, `Dispatch.Commission`, and `Dispatch.Story`. Expected counts are at least 6, 4, 10, and 37 passing tests respectively, with zero failures.

---

### Task 3: Final Verification and Blueprint Handoff

**Files:**
- Inspect all changed C++ and test files.
- Do not modify `Content/BluePrints/Widget/WBP_LetterUI.uasset` or other assets.

**Interfaces:**
- Consumes: completed widget and mailbox APIs.
- Produces: verified build and exact two-step Blueprint migration instructions.

- [ ] **Step 1: Build `DispatchEditor`**

Run Unreal Build Tool for the editor target. Expected: `Result: Succeeded`.

- [ ] **Step 2: Run complete affected automation suites**

Expected: every Inventory, Mailbox, Commission, and Story test passes.

- [ ] **Step 3: Inspect the diff**

Run `git diff --check`, `git status --short`, and confirm no new Codex-authored `.uasset` changes. Existing user asset modifications must remain untouched.

- [ ] **Step 4: Handoff**

Tell the user to reparent `WBP_LetterUI` to `LetterWidgetBase`, set `BP_Mailbox.LetterWidgetClass` to `WBP_LetterUI`, compile both assets, and then remove the obsolete Blueprint close and `SetLetterInfo` graph logic.
