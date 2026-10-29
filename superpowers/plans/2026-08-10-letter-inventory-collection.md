# Letter Inventory Collection Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Deliver `ULetterDefinition` assets through the mailbox, collect them into the existing Items inventory after reading, and expose the C++ data needed for a future inventory `Read` button.

**Architecture:** `ULetterDefinition` is the content source and creates a non-stackable `FDispatchItemInfo` carrying an optional back-reference to the definition. `AMailbox` owns delivery/read completion; `APlayerCharacter` owns idempotent collection and lookups. The existing DataTable/ID path remains as a deprecated fallback while current Blueprint assets are migrated.

**Tech Stack:** Unreal Engine 5.6 C++, UObject reflection, StateTree native tasks, UE Automation Tests.

## Global Constraints

- Do not modify `.uasset` files.
- Do not implement the inventory button, accessories, equipment slots, or accessory effects.
- Preserve current material-item and legacy DataTable behavior.
- Do not create commits unless the user explicitly requests them.

---

### Task 1: Letter inventory representation

**Files:**
- Modify: `Source/Dispatch/Public/Items/Item.h`
- Modify: `Source/Dispatch/Public/Quest/LetterDefinition.h`
- Modify: `Source/Dispatch/Private/Quest/LetterDefinition.cpp`
- Create: `Source/Dispatch/Private/Tests/LetterInventoryTests.cpp`

**Interfaces:**
- Produces: `FDispatchItemInfo::LetterDefinition`
- Produces: `ULetterDefinition::IsDefinitionValid() const`
- Produces: `ULetterDefinition::MakeInventoryItemInfo() const`
- Produces: `AItem::GetLetterDefinitionFromItemInfo(const FDispatchItemInfo&)`

- [ ] Write an automation test that creates a letter definition and expects ID/title/summary/icon mapping, count `1`, stack limit `1`, and the source definition reference.
- [ ] Build `DispatchEditor`; expect RED because the new fields and functions do not exist.
- [ ] Add `InventoryIcon`, multiline `InventorySummary`, validation, conversion, and a Blueprint-pure item-info lookup helper.
- [ ] Build and run `Dispatch.Inventory.LetterDefinition`; expect PASS.

Core conversion:

```cpp
FDispatchItemInfo ULetterDefinition::MakeInventoryItemInfo() const
{
    FDispatchItemInfo Item;
    Item.ItemID = LetterID;
    Item.ItemName = Title;
    Item.Description = InventorySummary;
    Item.Icon = InventoryIcon;
    Item.Count = 1;
    Item.MaxStack = 1;
    Item.LetterDefinition = const_cast<ULetterDefinition*>(this);
    return Item;
}
```

### Task 2: Player collection and commission safety

**Files:**
- Modify: `Source/Dispatch/Public/Characters/PlayerCharacter.h`
- Modify: `Source/Dispatch/Private/Characters/PlayerCharacter.cpp`
- Modify: `Source/Dispatch/Private/Commission/CommissionSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/InventoryTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/CommissionSubsystemTests.cpp`

**Interfaces:**
- Consumes: `ULetterDefinition::MakeInventoryItemInfo()`
- Produces: `bool APlayerCharacter::CollectLetter(const ULetterDefinition*)`
- Produces: `ULetterDefinition* APlayerCharacter::FindCollectedLetter(FName) const`
- Produces: `int32 APlayerCharacter::GetConsumableItemCount(FName) const`

- [ ] Add failing tests for valid collection, duplicate idempotency, lookup, invalid rejection, and letter exclusion from item transactions/preparation lists.
- [ ] Run focused Inventory/Commission tests; expect RED on missing APIs or unwanted letter availability.
- [ ] Implement collection and separate total count from consumable-material count.
- [ ] Make commission list/availability and inventory transaction removal ignore entries with a valid `LetterDefinition`.
- [ ] Run focused Inventory/Commission tests; expect PASS.

Collection contract:

```cpp
bool APlayerCharacter::CollectLetter(const ULetterDefinition* Letter)
{
    if (!IsValid(Letter) || !Letter->IsDefinitionValid()) return false;
    if (FindCollectedLetter(Letter->LetterID)) return true;
    AddItem(Letter->MakeInventoryItemInfo());
    return FindCollectedLetter(Letter->LetterID) == Letter;
}
```

### Task 3: Mailbox direct delivery and read completion

**Files:**
- Modify: `Source/Dispatch/Public/Items/Mailbox.h`
- Modify: `Source/Dispatch/Private/Items/Mailbox.cpp`
- Modify: `Source/Dispatch/Private/Tests/MailboxTests.cpp`

**Interfaces:**
- Consumes: `APlayerCharacter::CollectLetter`
- Produces: `bool AMailbox::DeliverLetter(ULetterDefinition*)`
- Produces: `bool AMailbox::CompleteLetterReading(APlayerCharacter*)`
- Produces: Blueprint event `OnLetterOpened(AActor*, ULetterDefinition*)`

- [ ] Add failing tests for direct delivery, invalid delivery preserving state, completion adding inventory, duplicate completion prevention, signal construction, and mailbox clearing.
- [ ] Run `Dispatch.Items.Mailbox`; expect RED.
- [ ] Store `CurrentLetter`, open it through the new Blueprint event on interaction, and defer collection/signal/clear until `CompleteLetterReading`.
- [ ] Retain `LetterDataTable`, `LetterItemInfo`, `DeliverLetterByID`, and `OnLetterTaken` as deprecated legacy fallback behavior.
- [ ] Run `Dispatch.Items.Mailbox`; expect PASS.

### Task 4: StateTree direct letter asset input

**Files:**
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryDeliverLetterTaskTests.cpp`

**Interfaces:**
- Consumes: `AMailbox::DeliverLetter`
- Produces: `FStoryStateTreeDeliverLetterTaskInstanceData::Letter`
- Retains: `FStoryStateTreeDeliverLetterTaskInstanceData::LetterID` fallback

- [ ] Add failing reflection tests that require a `ULetterDefinition` object property and retained legacy `LetterID`.
- [ ] Build; expect RED because the direct asset property is absent.
- [ ] Prefer `Letter` in `EnterState`; use `DeliverLetterByID` only when the direct asset is null and `LetterID` is present.
- [ ] Run `Dispatch.Story.DeliverLetter` and `Dispatch.Story.Flow.StateTree`; expect PASS.

### Task 5: Full verification

**Files:**
- Review all modified C++ and test files.

- [ ] Run `git diff --check` and inspect the complete diff.
- [ ] Build `DispatchEditor Win64 Development`.
- [ ] Run `Dispatch.Inventory`, `Dispatch.Items.Mailbox`, `Dispatch.Commission`, and `Dispatch.Story` automation suites.
- [ ] Confirm no `.uasset` changed during implementation and report the later Blueprint migration steps separately.
