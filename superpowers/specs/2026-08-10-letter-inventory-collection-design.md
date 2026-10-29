# Letter Inventory Collection Design

## Goal

Replace the mailbox letter DataTable path with `ULetterDefinition` assets. A letter is shown when the mailbox is opened, collected into the existing Items inventory only after the reading UI closes, and can later be reopened through a generic primary-action button on the inventory detail page.

## Scope

- C++ support for delivering a `ULetterDefinition` directly to the mailbox.
- Inventory presentation fields on `ULetterDefinition`.
- Conversion from a letter definition to one non-stackable inventory entry.
- Idempotent collection: the same `LetterID` cannot create duplicate entries.
- Blueprint-callable lookup needed by the future inventory `Read` button.
- Letters are excluded from commission consumable-material lists and transactions.
- Existing DataTable/`LetterID` delivery remains as a deprecated compatibility fallback until the current Blueprint and StateTree assets are migrated.

This change does not implement the inventory button, accessory equipment, wearable slots, accessory effects, or Widget Blueprint layout.

## Data Model

`ULetterDefinition` remains the single source of truth and gains:

- `InventoryIcon`
- `InventorySummary`
- `MakeInventoryItemInfo()`

The generated `FDispatchItemInfo` uses:

- `ItemID = LetterID`
- `ItemName = Title`
- `Description = InventorySummary`
- `Icon = InventoryIcon`
- `Count = 1`
- `MaxStack = 1`
- `LetterDefinition = this`

`FDispatchItemInfo` gains an optional `LetterDefinition` reference. A null reference means an ordinary material item. This avoids introducing an accessory type before an accessory system exists.

## Runtime Flow

1. `Story: Deliver Letter` receives a `ULetterDefinition` asset and calls `AMailbox::DeliverLetter`.
2. The mailbox stores it as the current letter and displays its available state.
3. Interaction invokes a Blueprint event with the definition so the existing letter UI can display it.
4. The future UI close handler calls `AMailbox::CompleteLetterReading`.
5. Completion adds the generated entry to `APlayerCharacter::InventoryItems`, submits `Story.Trigger.MailOpened` with `ContextName = LetterID`, and clears the mailbox.
6. The inventory detail page can resolve the selected entry's `LetterDefinition`; its future generic primary-action button will display `Read` and reopen the letter UI.

## Compatibility

The current `DT_Letters`, `DeliverLetterByID`, `LetterItemInfo`, and the StateTree `LetterID` input stay available as deprecated fallback APIs. New content uses direct `ULetterDefinition` references. This lets the user reconnect Blueprint and StateTree assets later without preventing the project from loading now.

## Error Handling

- Null or invalid letter definitions are rejected without changing mailbox state.
- A definition without `LetterID` or `Title` cannot be delivered or collected.
- Duplicate collection is treated as an idempotent success and remains one inventory entry.
- Completing a read without a current letter or valid player fails without sending a signal.
- Letter entries are rejected as commission consumables even if a caller supplies their ID directly.

## Verification

- Automation tests for letter-to-inventory conversion.
- Automation tests for idempotent collection and definition lookup.
- Mailbox tests for direct delivery and read completion.
- StateTree reflection tests for the new direct asset input and retained legacy fallback.
- Commission tests showing letters are excluded/rejected.
- Build `DispatchEditor` and run the relevant Inventory, Mailbox, Story, and Commission suites.
