# DA-Only Letter Flow Design

## Goal

Use `ULetterDefinition` data assets as the only source of letter content and delivery data. Remove the legacy DataTable/row-ID path so designers cannot configure two competing inputs on the same StateTree task.

## Runtime Design

- `FStoryStateTreeDeliverLetterTaskInstanceData` exposes only `Letter` (`ULetterDefinition*`).
- `FStoryStateTreeDeliverLetterTask::EnterState` fails when `Letter` is invalid and otherwise calls `AMailbox::DeliverLetter`.
- `AMailbox` stores only `CurrentLetter`; it no longer exposes `LetterItemInfo`, `LetterDataTable`, or `DeliverLetterByID`.
- `bHasLetter` becomes Blueprint read-only and `SetHasLetter` is no longer a Blueprint API, preventing an occupied mailbox state without a definition.
- Reading a mailbox letter still calls `CompleteLetterReading`, adds the definition-backed inventory entry once, clears the mailbox, and submits the existing `Story.Trigger.MailOpened` signal.
- Inventory re-reading remains presentation-only and does not collect a duplicate.

## Asset Migration

- Remove the obsolete `OnLetterTaken` Blueprint event branch from `BP_Mailbox`; retain the new definition-based `OnLetterOpened` hook.
- Clear the `DT_Letters` reference stored on the placed mailbox actor in `/Game/Maps/Demo`.
- Resave `ST_Day01` after the C++ schema change so its serialized legacy `LetterID` input field is discarded while its `DA_Letter_Day01_Merchant` reference remains. The identifier value `Letter_Day01_Intro` remains valid as the DA's identity and mail-opened signal context; it is no longer a second input.
- Delete `/Game/BluePrints/DataTable/DT_Letters`. The file is tracked by Git and can be restored through version control if required.

## Removed API

- `FMailboxLetterRow`
- `AMailbox::LetterItemInfo`
- `AMailbox::LetterDataTable`
- `AMailbox::DeliverLetterByID`
- Blueprint-callable `AMailbox::SetHasLetter`
- `AMailbox::OnLetterTaken`
- `FStoryStateTreeDeliverLetterTaskInstanceData::LetterID`

## Verification

- Reflection tests require all removed properties/functions/structs to be absent.
- Asset tests require the DT package and its package referencers to be absent.
- Direct delivery, mailbox reading, inventory collection/re-reading, commission validation, and Story StateTree suites must pass.
- Build `DispatchEditor`, run `git diff --check`, and verify no unrelated `.uasset` is changed by the migration.

## Constraints

- Work in the current checkout without staging or committing.
- Preserve all unrelated user-authored Blueprint and art changes.
- Modify only the exact legacy mailbox graph branch and exact letter-related asset references described above.
