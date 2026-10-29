# DATA-FOUNDATION-001 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: use `superpowers:executing-plans` to execute this plan task-by-task in the current local checkout. Do not create a worktree. Each production change follows RED → GREEN → regression verification.

**Goal:** Make the Dispatch Demo's authored narrative content editable through formal DataAssets and Story Workspace, with one Catalog authority, readable validation/migration diagnostics, isolated formal-DA preview, and a replaceable end-to-end data sample.

**Architecture:** Preserve the existing Campaign/Catalog/Episode and gameplay subsystems. Introduce compatibility boundaries around legacy DataTables and mixed structs, then migrate author-facing references toward typed soft DataAsset references without moving runtime state into content assets. Preview duplicates selected formal assets into an isolated transient session; runtime subsystems remain the only owners of play-state.

**Tech Stack:** Unreal Engine 5.6, C++20/UHT reflection, Primary DataAssets, Dispatch/DispatchEditor modules, Unreal Automation Tests, editor commandlets/services.

**Spec:** `docs/narrative/FOUNDATION_DATA_DESIGN.md` implementing `docs/narrative/DEMO_DESIGN.md`; progress authority is `docs/narrative/IMPLEMENTATION_MATRIX.md`.

## Global Constraints

- Use `F:\Epic Games\Unreal Projects\Dispatch` directly; do not create a worktree.
- Preserve existing tracked and untracked work; do not reset, clean, or overwrite user assets.
- `DATA-FOUNDATION-001` is the only main implementation subtask while this plan runs.
- Do not implement NPC movement, anchors, schedules, activity StateTrees, or formal seven-day prose.
- Do not delete legacy DataTables or compatibility APIs until a reference-zero report and user-approved migration acceptance exist.
- Do not save formal `.uasset` files until the migration/preview code and dry-run diagnostics are GREEN.
- Update `STATUS.md`, `IMPLEMENTATION_MATRIX.md`, and the task handoff after each verifiable batch.
- Do not commit or push unless the user separately authorizes it.

---

### Task 1: Restore a verifiable baseline and introduce pure Commission presentation data

**Files:**
- Modify: `Source/Dispatch/Public/Commission/CommissionDefinition.h`
- Modify: `Source/Dispatch/Private/Commission/CommissionDefinition.cpp`
- Modify: `Source/Dispatch/Private/Tests/CommissionDataAssetTests.cpp`
- Modify only as required by compiler/runtime callers: Commission/NoticeBoard/Widget source that reads `Definition.Display`

**Interfaces:**
- Produces: `FCommissionPresentation`, `UCommissionDefinition::Presentation`, and one compatibility accessor that prefers formal presentation and falls back to legacy `Display`.
- Preserves: serialized legacy `Display` and its readers during migration; runtime `EQuestRuntimeState` remains outside the new presentation struct.

- [x] Write a reflection/behavior test proving `FCommissionPresentation` contains title, summary, description, meta, location and images but no `QuestState`.
- [x] Write a compatibility test proving a legacy-only Definition still produces the same visible presentation.
- [x] Write a formal-data test proving `Presentation` wins when both new and legacy values exist.
- [x] Build and run the focused tests; verify the new tests fail only because the new type/API is absent.
- [x] Add the minimum pure presentation struct and compatibility accessor; do not remove legacy serialized data.
- [x] Route Commission cards, notice conversion and preview display through the accessor.
- [x] Rebuild and run the directly affected Commission, CommissionDispatch, NoticeBoard and Story Workspace Commission regressions. The broader `Dispatch.Widget.Commission` baseline has three unrelated pre-existing selection/MovieScene failures recorded in STATUS and handoff.
- [x] Record exact build/test counts and update matrix/STATUS/handoff.

### Task 2: Add direct Dialogue Choice effects through the shared narrative rule adapter

**Files:**
- Modify: `Source/Dispatch/Public/Quest/DispatchDialogueTypes.h`
- Modify: `Source/Dispatch/Public/Quest/DialogueDefinition.h`
- Modify: `Source/Dispatch/Private/Quest/DialogueSubsystem.cpp`
- Audit: `Source/Dispatch/Public/Story/Rules/StoryRuleTypes.h` (no change required; the existing DialogueChoice gameplay bridge remains the shared Narrative adapter)
- Modify: corresponding Dialogue/Narrative automation tests.

**Interfaces:**
- Produces: authored effects directly on a choice; a conversion/evaluation boundary into existing narrative condition/effect semantics.
- Preserves: existing node effects and legacy dialogue condition/effect assets.

- [x] Add a test where selecting a choice applies its own effect exactly once without a hidden intermediary node.
- [x] Add validation tests for invalid choice effect references and duplicate choice IDs.
- [x] Run focused Dialogue tests and confirm correct RED behavior: DispatchEditor compile exits 1 only because `FDialogueChoiceOption::Effects` is absent.
- [x] Add the smallest serializable choice-effect field and apply it in the existing committed-choice transaction.
- [x] Reuse the existing DialogueChoice gameplay bridge as the Narrative adapter and the existing Dialogue effect executor; no second runtime fact ledger or branch evaluator was added.
- [x] Run Dialogue, GameplayBridge, Narrative Contract, and ActionExecutor Dialogue regressions. Direct and shared paths are GREEN; broad Dialogue is 12/13 with the isolated legacy day-zero OpenOption fixture recorded.
- [x] Update project records with new compatibility boundaries and evidence.

### Task 3: Replace author-facing bare content IDs with typed soft references where migration is unambiguous

**Files:**
- Modify: `Source/Dispatch/Public/Commission/CommissionTypes.h`
- Modify: `Source/Dispatch/Public/Commission/CommissionDefinition.h`
- Modify: `Source/Dispatch/Public/Quest/LetterDefinition.h`
- Modify: `Source/Dispatch/Public/Story/Rules/StoryRuleTypes.h` / Day writing types as required
- Modify: matching resolver and validation implementations/tests.

**Interfaces:**
- Produces: typed references for Commission result knowledge/presentation, Letter clue grants, and Day-authored content.
- Preserves: stable sub-IDs and legacy bare-ID fallback until migration reports zero live references.

- [x] Write tests proving typed references resolve by asset identity; compile-time types reject wrong asset classes and validation rejects null formal entries.
- [x] Write fallback tests proving legacy IDs/hard references still load without becoming the preferred authoring path.
- [x] Verify focused RED, implement one content relationship at a time, and return each relationship to GREEN before the next.
- [x] Ensure runtime snapshots retain stable content ID plus source asset/instance identity.
- [x] Run Commission, Clue, Letter, Writing, Catalog and Narrative data-contract regressions.
- [x] Update the migration matrix in the local data design with the additive-property serialization constraint.

### Task 4: Make Catalog validation field-addressable and complete for direct references

**Files:**
- Modify: `Source/Dispatch/Public/Story/Content/StoryContentDefinition.h`
- Modify: `Source/Dispatch/Public/Story/Content/StoryContentCatalog.h`
- Modify: `Source/Dispatch/Private/Story/Content/StoryContentCatalog.cpp`
- Modify: each Definition implementation whose direct references are incomplete
- Modify: `Source/Dispatch/Private/Tests/StoryContentCatalogTests.cpp`
- Modify: Story Workspace validation presentation if required.

**Interfaces:**
- Produces: diagnostics containing owning asset path, field/sub-element path, referenced asset path, and reason.
- Covers: null/load failure, duplicate content ID, wrong type, unregistered reference, invalid child definition, duplicate inference pair, missing Commission default outcome, invalid Day writing shape.

- [x] Write failing tests for field-addressable diagnostics and missing direct references from Commission, Dialogue, Letter and Day.
- [x] Add a structured diagnostic record while retaining the current `TArray<FText>` compatibility output.
- [x] Enumerate direct references from all seven formal content types.
- [x] Display readable diagnostics in Story Workspace without creating another validation engine.
- [x] Run Catalog, StoryContentDefinition, Narrative Contract and Story Workspace validation regressions.
- [x] Update matrix/STATUS/handoff with the exact diagnostic coverage.

### Task 5: Add a read-only legacy content inventory and migration report

**Files:**
- Create: focused DispatchEditor migration report service/types under `Source/DispatchEditor/Public|Private/StoryWorkspace/`
- Modify: Story Workspace validation/migration panel or command entry
- Add: DispatchEditor automation tests for dry-run scanning.

**Interfaces:**
- Produces: a report row with legacy source asset/table, row/property path, target content type, resolved formal asset if any, status, and diagnostic.
- Never mutates or saves assets in default/dry-run mode.

- [x] Write tests using transient DataTables and Definitions for mapped, missing, ambiguous and already-migrated rows.
- [x] Confirm RED for missing report service.
- [x] Implement scanners for Quest, Dialogue, Clue, Document, CommissionRules and Report/Writing configuration references.
- [x] Add deterministic summary counts and a reference-zero gate; do not add deletion behavior.
- [x] Run migration report and Story Workspace tests, then execute a read-only scan of the project content.
- [x] Save the report under `Saved/StoryMigration/` and record its path/counts in project docs.

### Task 6: Make isolated Commission preview formal-DA-only

**Files:**
- Modify: `Source/DispatchEditor/Public|Private/StoryWorkspace/StoryCommissionPreviewSession.*`
- Remove production use of: `StoryCommissionPreviewData.*` hard-coded authored seeds (tests may retain transient fixtures)
- Modify: `StoryUIPreviewCoordinator.*`, validation UI, and preview tests.

**Interfaces:**
- Consumes: selected formal `UCommissionDefinition` and `UNPCDefinition` references.
- Produces: duplicated transient snapshots and isolated runtime state; source DA, savegame and current map remain unchanged.

- [x] Change tests to require at least one selected formal Commission/NPC pair and reject fallback C++ prose.
- [x] Verify RED while fallback preview seeds still open successfully.
- [x] Remove fallback authored data from the production preview path.
- [x] Prove source properties/package dirty state are unchanged after preview initialization, mutation and reset.
- [x] Run Commission preview data/session/coordinator and relevant PIE presentation tests.
- [x] Update docs with the user-visible preview selection flow and errors.

### Task 7: Generate the replaceable formal-DA sample through a guarded editor tool

**Files:**
- Create: guarded DispatchEditor sample builder/commandlet and tests
- Create on explicit apply only: sample assets under a dedicated developer/sample Content folder
- Modify: Story Workspace entry for selecting/validating the sample if required.

**Interfaces:**
- Dry run: validates destination paths, class compatibility, collision policy and reference graph without saving.
- Apply: creates one Campaign, one Episode, one Day, two NPCs, one Commission, two Clues, one Inference, two result Dialogues, one Letter, and three writing sections.

- [x] Test dry-run graph shape and collision refusal entirely with transient packages.
- [x] Test that the sample supports default dispatch without investigation and an alternate result after inference/preparation.
- [x] Implement dry-run first and run it against the intended destination.
- [x] Only after dry-run GREEN and status recording, apply asset creation once with backups/collision refusal.
- [x] Reload assets in a fresh editor process and run Catalog validation plus isolated Commission preview.
- [x] Record exact asset paths and keep all sample prose explicitly replaceable/development-only.

### Task 8: Completion gate and manual acceptance

**Files:**
- Update: `docs/narrative/IMPLEMENTATION_MATRIX.md`
- Update: `docs/narrative/STATUS.md`
- Create/update: `docs/narrative/handoffs/2026-09-14-data-foundation-001.md`

**Interfaces:**
- Produces: fresh build/test evidence, migration report, formal sample asset list, and a manual acceptance checklist.

- [x] Run fresh `DispatchEditor Win64 Development` build.
- [x] Run focused suites for Commission, Dialogue, Catalog, Migration, Preview, Writing, Clue/Inference and Narrative contracts.
- [x] Run the replaceable sample through the formal-DA isolated preview in a fresh process.
- [ ] Verify `git diff --check`, tracked/untracked scope, and that no unrelated Content asset changed.
- [ ] Ask the user to edit sample text/image/reference and confirm the preview reflects it; record manual result separately from automation.
- [x] Mark only matrix cells supported by current-workspace evidence; leave formal seven-day content, map performance and final Demo acceptance open.

## Execution Order and Stop Conditions

Tasks execute strictly in order because later migrations and sample assets depend on earlier serialization, reference and validation contracts. Stop immediately and update STATUS/matrix/handoff if UHT reveals an unsafe serialized-property change, a migration is ambiguous, a formal asset would be overwritten, or new code affects a previously GREEN requirement without a fresh regression. A task is not complete until it has an observable editor/preview/diagnostic result in addition to internal types and tests.
