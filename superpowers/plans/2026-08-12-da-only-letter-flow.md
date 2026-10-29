# DA-Only Letter Flow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove the legacy DataTable/letter-ID delivery path and make `ULetterDefinition` the only configurable letter source.

**Architecture:** StateTree passes one `ULetterDefinition` directly to `AMailbox`. The mailbox opens and collects that definition; no runtime component performs DataTable lookup or carries a duplicate `FDispatchItemInfo` payload.

**Tech Stack:** Unreal Engine 5.6 C++, StateTree, UMG, Blueprint asset migration, Unreal Automation Tests.

## Global Constraints

- Work in the current checkout without staging or committing.
- Preserve all unrelated user-authored Blueprint and art changes.
- Delete only `Content/BluePrints/DataTable/DT_Letters.uasset`.
- Modify only the obsolete `OnLetterTaken` graph branch, the placed Demo mailbox's `LetterDataTable` property, and the stale `LetterID` serialization in `ST_Day01`.

---

### Task 1: Define the DA-Only Contract with Failing Tests

**Files:**
- Modify: `Source/Dispatch/Private/Tests/StoryDeliverLetterTaskTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/MailboxTests.cpp`

**Interfaces:**
- Produces: reflection requirements that `LetterID`, `LetterDataTable`, `LetterItemInfo`, `DeliverLetterByID`, Blueprint-callable `SetHasLetter`, `OnLetterTaken`, and `MailboxLetterRow` do not exist.

- [ ] Change the Story responsibility test to require `LetterID` to be absent.
- [ ] Add mailbox reflection assertions for every removed property, function, event, and row struct.
- [ ] Add an editor asset assertion that `/Game/BluePrints/DataTable/DT_Letters` does not exist and has no package referencers.
- [ ] Build and run the focused tests; confirm RED because the legacy contract and asset still exist.

### Task 2: Migrate Referencing Assets

**Files:**
- Modify: `Content/BluePrints/Items/Mailbox/BP_Mailbox.uasset`
- Modify: the exact external Demo mailbox actor package that references `DT_Letters`

**Interfaces:**
- Consumes: existing `OnLetterTaken` event and `LetterDataTable` property before C++ removes them.
- Produces: assets that compile and serialize without the legacy branch/reference.

- [ ] Remove the isolated `OnLetterTaken` execution branch from `BP_Mailbox` while leaving `OnLetterOpened` and unrelated graph nodes untouched.
- [ ] Clear `LetterDataTable` on the placed `BP_Mailbox` actor in `/Game/Maps/Demo`.
- [ ] Compile/save `BP_Mailbox` and save only the dirty external actor package.
- [ ] Verify binary dependency scanning no longer finds a `DT_Letters` reference outside the DT asset itself.

### Task 3: Remove Legacy C++ and the DT Asset

**Files:**
- Modify: `Source/Dispatch/Public/Items/Mailbox.h`
- Modify: `Source/Dispatch/Private/Items/Mailbox.cpp`
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Delete: `Content/BluePrints/DataTable/DT_Letters.uasset`

**Interfaces:**
- Retains: `bool AMailbox::DeliverLetter(ULetterDefinition*)` and `ULetterDefinition* Letter`.
- Removes: all APIs enumerated in Task 1.

- [ ] Remove the DataTable include, row struct, duplicate item payload, DataTable property, legacy function, and obsolete event from `AMailbox`.
- [ ] Make `bHasLetter` Blueprint read-only and keep state mutation internal to delivery/read completion.
- [ ] Simplify mailbox interaction to the `CurrentLetter` path only.
- [ ] Remove `LetterID` from the StateTree instance data and fail clearly when `Letter` is unset.
- [ ] Delete the exact tracked `DT_Letters.uasset` file after verifying its resolved path is inside the project.
- [ ] Build and run the focused tests; confirm GREEN.

### Task 4: Resave StateTree and Verify the Migration

**Files:**
- Modify: `Content/BluePrints/Story/StateTree/Day01/ST_Day01.uasset`

**Interfaces:**
- Produces: `ST_Day01` serialized with only `DA_Letter_Day01_Merchant` for its delivery task.

- [ ] Resave `ST_Day01` through UnrealEditor after the schema change so unknown legacy property data is stripped.
- [ ] Verify the StateTree binary retains the DA path and contains no serialized `LetterID` input field; retain `Letter_Day01_Intro` as the DA/signal identity value.
- [ ] Run `Dispatch.Items.Mailbox`, `Dispatch.Story`, `Dispatch.Inventory`, `Dispatch.UI.LetterWidget`, and `Dispatch.Commission` with zero failures.
- [ ] Build `DispatchEditor`, run `git diff --check`, inspect final asset scope, and confirm the DT file is the only deleted asset authored by this migration.
