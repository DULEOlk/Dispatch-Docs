# Unified Commission Definition and Outcome Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build one deterministic `CommissionID`-driven path from quest content and dispatch rules through draft preparation, NPC assignment, locked outcomes, idempotent settlement, dialogue/scene/story effects, and a C++-owned three-slot UI.

**Architecture:** Add focused commission types plus a pure rule evaluator, then place all mutable cross-map state and orchestration in `UCommissionSubsystem`. Keep `UNPCCommissionComponent` as the NPC capability/occupancy boundary, let `UDialogueUIControllerSubsystem` own the modal hand-off between the existing dialogue screen and commission widget, and call existing inventory, clue, world-state, story-director, dialogue, and NPC-scene interfaces at transaction boundaries.

**Tech Stack:** Unreal Engine 5.6, C++17, Gameplay Tags, UGameInstanceSubsystem, ULocalPlayerSubsystem, UMG, DataTable, Unreal Automation Framework.

## Global Constraints

- Preserve every existing uncommitted or committed user change; never reset, clean, overwrite, or delete unrelated work.
- Widget Blueprints provide layout/art only; C++ resolves data, owns draft state, evaluates rules, mutates inventory/NPC state, and starts result presentation.
- Preserve `FQuestContentRow`, report-writing `FQuestDefinition`, `FNoticeQuest`, `FQuestInfo`, the old `UNPCCommissionComponent` overloads, `ANPCCharacter::OnRequestCommissionUI`, and the existing NPC dialogue UI/session chain.
- Use `FQuestContentRow.DispatchRuleID`; do not add another quest-to-commission link.
- Do not put commission conditions/outcomes in StateTree, add randomness, build scheduling/Behavior Tree systems, or redesign report writing.
- The first increment supplies complete Demo-capable code and schemas only. `DT_CommissionRules`, rule rows, NPC tags, and Widget Blueprint bindings remain editor-authored content.
- Empty optional return dialogue, result letter, follow-up scene, named result, or story signal effects are valid. Configured invalid references fail safely and log commission/rule/NPC/instance/outcome/stage context.
- All production behavior follows red-green TDD; every new behavior is first observed failing for the intended reason.

## File Map

- Create `Source/Dispatch/Public/Commission/CommissionTypes.h`: reflected definitions, conditions, outcomes, rules, draft, instance, resolved definition, evaluation, and UI preparation view data.
- Create `Source/Dispatch/Public/Commission/CommissionRuleEvaluator.h` and `Source/Dispatch/Private/Commission/CommissionRuleEvaluator.cpp`: structural validation and pure deterministic matching only.
- Create `Source/Dispatch/Public/Commission/CommissionSubsystem.h` and `Source/Dispatch/Private/Commission/CommissionSubsystem.cpp`: resolution, runtime validation, draft/session state, preparation aggregation, dispatch transaction, instances, and settlement.
- Create `Source/Dispatch/Public/Widget/CommissionPreparationViewModel.h` and `Source/Dispatch/Private/Widget/CommissionPreparationViewModel.cpp`: C++ list-entry data for mixed preparation choices.
- Modify `Source/Dispatch/Public/Quest/DispatchContentSettings.h`, `Source/Dispatch/Private/Quest/DispatchContentSettings.cpp`, and `Config/DefaultGame.ini`: commission rules and commission widget settings entries.
- Modify `Source/Dispatch/Public/Quest/QuestContentSubsystem.h` and `Source/Dispatch/Private/Quest/QuestContentSubsystem.cpp`: resolve the authoritative quest row/dispatch-rule link without changing report writing.
- Modify `Source/Dispatch/Public/Characters/PlayerCharacter.h` and `Source/Dispatch/Private/Characters/PlayerCharacter.cpp`: query and atomically mutate inventory counts.
- Modify `Source/Dispatch/Public/Component/NPCCommissionComponent.h` and `Source/Dispatch/Private/Component/NPCCommissionComponent.cpp`: tags, instance snapshot, new assignment path, and legacy compatibility.
- Modify `Source/Dispatch/Public/Characters/NPCCharacter.h` and `Source/Dispatch/Private/Characters/NPCCharacter.cpp`: accept an instance through the existing character state/routine notification boundary.
- Modify `Source/Dispatch/Public/Widget/UCommissionUIBase.h` and `Source/Dispatch/Private/Widget/UCommissionUIBase.cpp`: fixed current commission/NPC view, three mixed slots, content list, reaction/error state, confirm/cancel, and optional BindWidget presentation.
- Modify `Source/Dispatch/Public/Quest/DialogueSubsystem.h` and `Source/Dispatch/Private/Quest/DialogueSubsystem.cpp`: show `OpenCommission` only for a valid pending dispatch and available NPC while retaining the special-action delegate.
- Modify `Source/Dispatch/Public/Widget/DialogueUIControllerSubsystem.h` and `Source/Dispatch/Private/Widget/DialogueUIControllerSubsystem.cpp`: hide/restore the existing dialogue widget and create/destroy the commission widget.
- Modify `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h` and `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp` only if a narrow reference-query seam is needed for preflight validation; keep `PlayNPCScene` unchanged as the presentation entry point.
- Add focused tests under `Source/Dispatch/Private/Tests/CommissionDefinitionTests.cpp`, `CommissionRuleEvaluatorTests.cpp`, `CommissionSubsystemTests.cpp`, and `CommissionDialogueUITests.cpp`; extend `InventoryTests.cpp`, `NPCSystemTests.cpp`, and `CommissionUIBaseTests.cpp` for compatibility contracts.

---

### Task 1: Commission schemas, settings entry, quest link, and structural validation

**Files:**
- Create: `Source/Dispatch/Public/Commission/CommissionTypes.h`
- Create: `Source/Dispatch/Public/Commission/CommissionRuleEvaluator.h`
- Create: `Source/Dispatch/Private/Commission/CommissionRuleEvaluator.cpp`
- Create: `Source/Dispatch/Private/Tests/CommissionDefinitionTests.cpp`
- Modify: `Source/Dispatch/Public/Quest/DispatchContentSettings.h`
- Modify: `Source/Dispatch/Private/Quest/DispatchContentSettings.cpp`
- Modify: `Source/Dispatch/Public/Quest/QuestContentSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/QuestContentSubsystem.cpp`
- Modify: `Config/DefaultGame.ini`

**Interfaces:**
- Produces: `ECommissionPreparationType`, `FCommissionPreparationRef`, `FCommissionItemAmount`, `FCommissionNamedResult`, `FCommissionBranchConditions`, `FCommissionOutcome`, `FCommissionBranchRule`, `FCommissionRuleSetRow`, `FResolvedCommissionDefinition`, `FCommissionEvaluationContext`, `FCommissionEvaluation`, `FCommissionDraft`, `FCommissionInstance`.
- Produces: `bool FCommissionRuleEvaluator::ValidateRuleSet(FName CommissionID, FName RuleSetID, const FCommissionRuleSetRow& RuleSet, FString& OutError)`.
- Produces: `bool UQuestContentSubsystem::ResolveQuestDispatchRuleID(FName QuestID, FName& OutRuleSetID) const`.
- Consumes: `FQuestContentRow.DispatchRuleID`, `UDispatchContentSettings::QuestTable`, DataTable row names.

- [ ] Add reflection and behavior tests that construct an in-memory `FQuestContentRow` table and `FCommissionRuleSetRow`, then assert: valid `QuestID -> DispatchRuleID`; missing quest; empty dispatch rule; `MaxPreparationSlots != 3`; empty/missing/conditional default outcome; duplicate `OutcomeID`; branch/outcome ID mismatch; empty preparation IDs; non-positive item counts; invalid gameplay tags.
- [ ] Build `DispatchEditor` and confirm the test translation unit fails because commission types, settings property, and quest-link resolver do not exist.
- [ ] Define the reflected types. `FCommissionPreparationRef::operator==` compares both `Type` and `ContentID`; `FCommissionOutcome` carries stable `OutcomeID`, result grade/text, optional dialogue/letter/scene IDs, item/clue/named-result/signal effects, and explicit booleans for optional NPC body/commission-state changes.
- [ ] Add `CommissionRulesTable` and `CommissionWidgetClass` to `UDispatchContentSettings`, plus `LoadCommissionRulesTable()`. Leave the config paths blank so editor asset creation remains manual.
- [ ] Add `ResolveQuestDispatchRuleID` without changing `ResolveQuestContent` or report-table logic.
- [ ] Implement structural validation with exact context-rich error strings and an unconditional-default test helper. Do not resolve world actors or external tables in the pure validator.
- [ ] Build and run `Automation RunTests Dispatch.Commission.Definition;Quit`; confirm all definition/structural validation assertions pass.

### Task 2: Deterministic rule evaluator

**Files:**
- Modify: `Source/Dispatch/Public/Commission/CommissionRuleEvaluator.h`
- Modify: `Source/Dispatch/Private/Commission/CommissionRuleEvaluator.cpp`
- Create: `Source/Dispatch/Private/Tests/CommissionRuleEvaluatorTests.cpp`

**Interfaces:**
- Produces: `static bool Evaluate(const FCommissionRuleSetRow& RuleSet, const FCommissionEvaluationContext& Context, FCommissionEvaluation& OutEvaluation, FString& OutError)`.
- Produces: `static bool Matches(const FCommissionBranchConditions& Conditions, const FCommissionEvaluationContext& Context)`.
- Consumes: exact NPC ID, NPC tags, trust/body state, selected preparations, owned clue IDs, and world tags supplied in the immutable context.

- [ ] Write literal table-driven tests for default/no preparation; Required/Any/Forbidden preparation semantics; exact allowed/forbidden NPCs; required/forbidden NPC tags; minimum trust; body state; clue/world tag conditions; higher priority; same-highest-priority ambiguity; array-order independence; and reaction text/locked outcome identity.
- [ ] Build and run `Dispatch.Commission.Evaluator`; confirm failure occurs at the missing evaluator implementation, not fixture setup.
- [ ] Implement condition matching as all-required/any-one/none-forbidden. Collect all matching branches, choose the unique maximum priority, and reject a tie; never select by array order.
- [ ] Validate before evaluation. If no branch matches, resolve only the configured unconditional default; return its index, outcome ID, copied outcome, and preview reaction in one `FCommissionEvaluation`.
- [ ] Run `Dispatch.Commission.Evaluator` twice, including once with branch order reversed; confirm every case passes.

### Task 3: Inventory transaction seam and NPC instance compatibility

**Files:**
- Modify: `Source/Dispatch/Public/Characters/PlayerCharacter.h`
- Modify: `Source/Dispatch/Private/Characters/PlayerCharacter.cpp`
- Modify: `Source/Dispatch/Private/Tests/InventoryTests.cpp`
- Modify: `Source/Dispatch/Public/Component/NPCCommissionComponent.h`
- Modify: `Source/Dispatch/Private/Component/NPCCommissionComponent.cpp`
- Modify: `Source/Dispatch/Public/Characters/NPCCharacter.h`
- Modify: `Source/Dispatch/Private/Characters/NPCCharacter.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCSystemTests.cpp`

**Interfaces:**
- Produces: `int32 APlayerCharacter::GetItemCount(FName ItemID) const`, `bool CanApplyItemTransaction(const TArray<FCommissionItemAmount>& Removed, const TArray<FCommissionItemAmount>& Granted) const`, and `bool ApplyItemTransaction(...)` that computes on a copy and commits once.
- Produces: `FGameplayTagContainer UNPCCommissionComponent::CommissionTags`, `FGuid ActiveCommissionInstanceID`, `FCommissionInstance ActiveCommissionInstance`, `bool AssignCommission(const FCommissionInstance&)`, and `bool ANPCCharacter::TryAssignCommissionInstance(const FCommissionInstance&)`.
- Preserves: `CanAcceptCommission(FQuestInfo)`, `CanAcceptCommission(FName)`, both old `AssignCommission` forms, `CurrentCommissionID`, `ActiveCommission`, and existing Blueprint-callable names.

- [ ] Extend inventory tests with count queries, exact multi-stack removal/grant, rejection of missing counts/invalid amounts, and a byte-for-byte unchanged inventory assertion after a rejected transaction.
- [ ] Extend NPC tests to assert tags are exposed, a valid instance assigns and snapshots its GUID/outcome/preparations, a second instance is rejected, and `ClearCommission` clears both new and legacy state while old overloads still work.
- [ ] Build the two suites and confirm missing methods/properties produce the expected red result.
- [ ] Implement inventory transactions by normalizing amounts by ID, rejecting `NAME_None`/non-positive counts, applying removals and grants to a local copy, removing zero-count entries, and assigning `InventoryItems` only after all operations succeed.
- [ ] Implement the instance assignment path once and route legacy overloads through a compatibility instance/legacy snapshot without deleting old fields. Keep `IsAvailable()` and `CommissionState` authoritative for occupancy.
- [ ] Run `Dispatch.Inventory` and `Dispatch.NPC.Components`; confirm new and legacy assertions pass.

### Task 4: Commission subsystem definition resolution, draft, and locked dispatch

**Files:**
- Create: `Source/Dispatch/Public/Commission/CommissionSubsystem.h`
- Create: `Source/Dispatch/Private/Commission/CommissionSubsystem.cpp`
- Create: `Source/Dispatch/Private/Tests/CommissionSubsystemTests.cpp`

**Interfaces:**
- Produces Blueprint-callable `SetPendingDispatchCommission`, `ResolveCommissionDefinition`, `BeginDraft`, `SetPreparationSlot`, `ClearPreparationSlot`, `EvaluateCurrentDraft`, `ConfirmDispatch`, `CancelDraft`, `MarkCommissionReady`, `ResolveCommission`, and instance queries.
- Produces native test/provider seams for quest/rule/dialogue/scene tables, player inventory, clue subsystem, world state, story director, and target NPC without adding gameplay-facing test cleanup methods.
- Consumes: the evaluator, quest content/link resolver, rule table, `ANPCCharacter`, inventory transaction seam, `UClueSubsystem`, and `UStoryWorldStateSubsystem`.

- [ ] Write tests that build one complete Demo-capable in-memory rule set with three outcomes and assert invalid commission/link/quest mismatch/external dialogue/scene/clue references fail with non-empty errors.
- [ ] Add draft tests for three mixed slots, empty slots, duplicate refs, disallowed types, invalid/unowned content, cancel-without-mutation, NPC invalidation, and live context re-evaluation.
- [ ] Add dispatch tests for unavailable NPC, a second assignment, missing preparation at confirmation, transaction rollback, saved preparation snapshot, generated instance GUID, and immediate `ResolvedOutcomeID` lock.
- [ ] Build/run `Dispatch.Commission.Subsystem`; confirm the first failure is the missing subsystem API.
- [ ] Resolve `FResolvedCommissionDefinition` from the authoritative quest row plus rule row; run pure validation, then validate configured clue/dialogue/scene references through existing tables. Validate selected Item IDs against live inventory and Clue/Evidence IDs against `DT_Clues` plus collection state.
- [ ] Keep the draft as a fixed three-element array with empty `ContentID` representing an open slot. Reject the same `(Type, ContentID)` in two occupied slots.
- [ ] Build the evaluation context from an immutable NPC info/tag snapshot, selected content, collected clues, and current world tags. Re-evaluate on every preview and again on confirm.
- [ ] Confirm by preflighting every condition and item removal, applying the inventory transaction, assigning the complete instance through `ANPCCharacter`, and rolling inventory back if assignment unexpectedly fails. Add the active instance only after both succeed.
- [ ] Run `Dispatch.Commission.Subsystem`; confirm all resolution/draft/dispatch tests pass, including outcome lock after later world-state changes.

### Task 5: C++-owned three-slot commission UI

**Files:**
- Create: `Source/Dispatch/Public/Widget/CommissionPreparationViewModel.h`
- Create: `Source/Dispatch/Private/Widget/CommissionPreparationViewModel.cpp`
- Modify: `Source/Dispatch/Public/Widget/UCommissionUIBase.h`
- Modify: `Source/Dispatch/Private/Widget/UCommissionUIBase.cpp`
- Modify: `Source/Dispatch/Private/Tests/CommissionUIBaseTests.cpp`

**Interfaces:**
- Produces: `InitializeCommissionDraft(ANPCCharacter*)`, `SelectPreparationSlot(int32)`, `ChoosePreparation(UCommissionPreparationViewModel*)`, `ClearPreparationSlot(int32)`, `RefreshDraftPresentation()`, `CancelCommission()`, and native cancelled/dispatched delegates.
- Produces optional bindings named `Text_CommissionTitle`, `Text_CommissionSummary`, `Text_NPCName`, `Text_PreviewReaction`, `Text_Error`, `List_AvailablePreparations`, `Button_PreparationSlot0..2`, `Text_PreparationSlot0..2`, `Button_Dispatch`, and `Button_Return`.
- Preserves legacy properties/events and `InitializeCommissionUI(TargetNPC, AvailableQuests)`, `SelectQuestByIndex`, and `ConfirmCommission` as compatibility surfaces.

- [ ] Replace source/reflection-only UI tests with observable tests: initialization displays current commission/NPC, exactly three slots mirror the subsystem draft, changing/clearing a slot changes reaction and dispatch-enabled state, data errors disable confirm, cancel clears draft, and successful confirm emits once.
- [ ] Build `Dispatch.Widget.CommissionUIBase`; confirm missing new API/bindings cause the red failure while legacy contract tests still compile.
- [ ] Implement the view model as presentation data only. Aggregate inventory Item entries plus collected `DT_Clues` entries, treating clue rows whose `CategoryID` is `Evidence` as Evidence and other rows as Clue.
- [ ] Bind optional widgets in `NativeConstruct`, update text/button enabled state directly in C++, and use Blueprint events only as optional visual notifications. No rule matching or mutation enters a Blueprint event graph.
- [ ] Keep legacy initialization/selection operational for old assets/tests; route `ConfirmCommission` to the subsystem when a unified draft is active and to the old component only for the legacy call path.
- [ ] Run `Dispatch.Widget.CommissionUIBase` and `Dispatch.NPC.Components`; confirm both unified and compatibility paths pass.

### Task 6: Dialogue `OpenCommission` modal hand-off

**Files:**
- Modify: `Source/Dispatch/Public/Quest/DialogueSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/DialogueSubsystem.cpp`
- Modify: `Source/Dispatch/Public/Widget/DialogueUIControllerSubsystem.h`
- Modify: `Source/Dispatch/Private/Widget/DialogueUIControllerSubsystem.cpp`
- Create: `Source/Dispatch/Private/Tests/CommissionDialogueUITests.cpp`

**Interfaces:**
- Consumes: `UCommissionSubsystem::CanOpenPendingCommissionForNPC`, `BeginDraft`, `UDispatchContentSettings::CommissionWidgetClass`, and the existing `OnSpecialInteractionRequested` event.
- Produces: controller-owned commission widget lifecycle and restoration/close behavior; no new NPC Blueprint creation event.

- [ ] Test `OpenCommission` filtering for: no pending ID, invalid dispatch link, unavailable NPC, no handler, and the valid pending+rule+NPC+handler case.
- [ ] Test the controller-facing lifecycle policy without requiring a Blueprint asset: open hides the dialogue widget, cancel restores the same NPC interaction menu/session, dispatch success closes the interaction session, and failure keeps commission state active for correction.
- [ ] Build/run `Dispatch.Commission.DialogueUI`; confirm it fails because pending-definition filtering and controller handlers are absent.
- [ ] Make `EvaluateInteractionOptions` consult the commission subsystem in addition to current NPC availability and delegate binding. Preserve every existing dialogue option and the auto-appended Exit behavior.
- [ ] Bind `UDialogueUIControllerSubsystem` to `OnSpecialInteractionRequested`; on `OpenCommission`, begin the draft, collapse (do not destroy) the existing dialogue widget, create the configured `UCommissionUIBase`, and bind native completion delegates.
- [ ] On cancel, destroy only the commission widget, cancel the draft, show/synchronize the existing dialogue widget, and retain current NPC/session. On success, destroy the commission widget and call `CloseDialogueSession()` to restore NPC state and end the interaction.
- [ ] Run `Dispatch.Commission.DialogueUI`, `Dispatch.Dialogue`, and `Dispatch.NPC`; confirm the existing dialogue UI chain remains green.

### Task 7: Idempotent settlement and story/presentation effects

**Files:**
- Modify: `Source/Dispatch/Public/Commission/CommissionSubsystem.h`
- Modify: `Source/Dispatch/Private/Commission/CommissionSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/CommissionSubsystemTests.cpp`
- Modify: `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h` and `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp` only for a read-only `HasNPCSceneDefinition(FName)` helper if DataTable preflight cannot stay inside the commission subsystem.

**Interfaces:**
- Consumes: locked `ResolvedOutcomeID`, `APlayerCharacter::ApplyItemTransaction`, `UClueSubsystem::CollectClue`, `UStoryWorldStateSubsystem::SetNamedResult`, `AStoryDirector::SubmitStorySignal`, `UDialogueSubsystem::StartStoryDialogueForNPC`, and `UNPCSceneSubsystem::PlayNPCScene`.
- Produces: `MarkCommissionReady` and exactly-once `ResolveCommission`; state transitions `Active -> ReadyToResolve -> Resolved` and `bOutcomeEffectsApplied=false -> true`.

- [ ] Add tests for non-ready rejection; locked outcome surviving changed world state; granted/consumed items; clue grants; named results; story signal source/context; NPC body/commission state; duplicate resolve; correct NPC dialogue context; correct scene ID; and presentation failure after core settlement without core replay.
- [ ] Build/run the result tests and observe expected failures before implementation.
- [ ] Preflight all core effects before mutation: instance/rule/outcome identity, inventory transaction, clue definitions, named-result keys/values, valid story tags and story director presence, and target NPC identity. Empty optional effects pass.
- [ ] Apply inventory once, collect clues, set named results, submit story signals with `SourceID=CommissionID`, `ContextName=OutcomeID`, update the NPC, set `bOutcomeEffectsApplied`, then set `Resolved`. Since all post-preflight core setters are deterministic/no-fail, no partial commit is exposed.
- [ ] Start return dialogue with the exact NPC. If both dialogue and scene exist, bind a one-shot dialogue-ended continuation for that dialogue/NPC before starting the scene; otherwise start the scene immediately. Log presentation failure but never clear the applied flag or reapply core effects.
- [ ] Re-run `Dispatch.Commission.Subsystem`, `Dispatch.Dialogue`, `Dispatch.NPC.Scene`, and `Dispatch.Story.Flow`; confirm exactly-once effects and existing presentation behavior pass.

### Task 8: Full verification and editor handoff

**Files:**
- Modify: `docs/superpowers/plans/2026-08-04-unified-commission-definition-outcome.md` only to check completed boxes if useful; do not create or edit binary assets.

**Interfaces:**
- Verifies every acceptance criterion in the formal spec and preserves manual editor ownership of DataTables/Widget assets.

- [ ] Run `git status --short --branch` and `git diff --check`; inspect every changed file and confirm no unrelated user file was reverted or overwritten.
- [ ] Build with `& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild`; require exit code 0.
- [ ] Run Unreal automation with `Automation RunTests Dispatch.Commission;Automation RunTests Dispatch.NPC;Automation RunTests Dispatch.Dialogue;Automation RunTests Dispatch.Quest;Automation RunTests Dispatch.Story.Flow;Quit` under `-unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache`; require zero failed tests.
- [ ] Run `Automation RunTests Dispatch.Inventory;Quit` because commission transactions extend inventory behavior; require zero failed tests.
- [ ] Start and quit the editor commandlet once with `-ExecCmds=Quit` to detect reflected class/property load errors.
- [ ] Re-read the formal spec line by line and report any editor-manual remainder explicitly: create/configure `DT_CommissionRules`, set `DispatchRuleID`, NPC tags, Demo branches/results/reactions, bind the commission widget class and optional C++ widget names, and author optional dialogue/scene content.

