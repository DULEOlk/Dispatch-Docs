# Dispatch Unified Narrative Content Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make Commission and Dialogue DataAssets the authoritative static content entry points, add clue/lead provenance, and move the once-per-day three-section writing lifecycle out of `QuestSubsystem` into `DailyWritingSubsystem`.

**Architecture:** Existing commission evaluation, inventory settlement, NPC state, story state, and dialogue presentation remain the runtime engines. New Primary DataAssets own complete commission/dialogue definitions and are passed directly by StateTree and Blueprint APIs. `UDailyWritingSubsystem` reads facts from the existing runtime owners and stores only draft/submission records, while `UDayDefinition` owns the three-section static template.

**Tech Stack:** Unreal Engine 5.6 C++, UPrimaryDataAsset/UDataAsset, UGameInstanceSubsystem, Gameplay Tags, StateTree, UMG Blueprint APIs, Unreal Automation Tests.

**Spec:** `C:/Users/DULEOLK/.codex/visualizations/2026/08/06/019fd933-cca8-7a52-aeaa-ae3639ae03f7/2026-08-18-dispatch-narrative-content-design.md`

## Global Constraints

- Do not write planning/design files into the repository `docs` directory.
- Preserve the user's existing changes to `WBP_PlayerNotebook`, `LetterWidgetBase`, and `LetterWidgetTests`.
- `ULetterDefinition` remains separate from commissions.
- `DT_Clues` and `DT_GuideComments` remain DataTables.
- New gameplay/static-content paths use direct asset references; ID/DataTable methods remain only as explicitly deprecated migration adapters.
- Daily writing has exactly three sections, exactly one unconditional default option per section, one submission per day, and no immediate gameplay rewards.
- Every production behavior is introduced by an automation test that is observed failing before implementation.

---

### Task 1: Commission DataAsset Authoritative Path

**Files:**
- Create: `Source/Dispatch/Public/Commission/CommissionDefinition.h`
- Create: `Source/Dispatch/Private/Commission/CommissionDefinition.cpp`
- Modify: `Source/Dispatch/Public/Commission/CommissionTypes.h`
- Modify: `Source/Dispatch/Public/Commission/CommissionSubsystem.h`
- Modify: `Source/Dispatch/Private/Commission/CommissionSubsystem.cpp`
- Modify: `Source/Dispatch/Public/Items/NoticeBoard.h`
- Modify: `Source/Dispatch/Private/Items/NoticeBoard.cpp`
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Test: `Source/Dispatch/Private/Tests/CommissionDataAssetTests.cpp`
- Test: `Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp`

**Interfaces:**
- Produces: `UCommissionDefinition`, `UCommissionSubsystem::SetPendingDispatchCommission(UCommissionDefinition*)`, `BeginDraft(UCommissionDefinition*, ANPCCharacter*)`, `ResolveCommissionDefinition(const UCommissionDefinition*, FResolvedCommissionDefinition&)`, `ANoticeBoard::SetCommissionSlot(int32, UCommissionDefinition*, EQuestState)`.
- Preserves: ID overloads as `DeprecatedFunction` migration adapters only.

- [ ] **Step 1: Write the failing asset/notice/StateTree tests**

```cpp
UCommissionDefinition* Definition = NewObject<UCommissionDefinition>();
Definition->CommissionID = TEXT("Commission.Test");
Definition->Display.Title = FText::FromString(TEXT("Test Commission"));
Definition->MaxPreparationSlots = 3;
Definition->DefaultOutcomeID = TEXT("Outcome.Default");
Definition->Branches.Add(MakeDefaultBranch(TEXT("Outcome.Default")));
TestTrue(TEXT("Complete commission DA validates"), Definition->ValidateDefinition(Error));
TestTrue(TEXT("Notice board accepts the direct asset"), Board->SetCommissionSlot(0, Definition, EQuestState::Available));
TestEqual(TEXT("Notice derives its stable ID"), Board->QuestList[0].QuestID, Definition->CommissionID);
TestNotNull(TEXT("StateTree exposes Commission"), FindFProperty<FObjectPropertyBase>(DataStruct, TEXT("Commission")));
TestNull(TEXT("StateTree no longer mixes QuestID"), FindFProperty<FNameProperty>(DataStruct, TEXT("QuestID")));
```

- [ ] **Step 2: Run the focused test command and verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: compilation fails because `UCommissionDefinition`, direct-asset overloads, and the StateTree property do not exist.

- [ ] **Step 3: Implement the minimal direct-asset path**

```cpp
UCLASS(BlueprintType)
class DISPATCH_API UCommissionDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FName CommissionID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FQuestInfo Display;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, meta=(ClampMin="3", ClampMax="3")) int32 MaxPreparationSlots = 3;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TArray<ECommissionPreparationType> AllowedPreparationTypes;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FName DefaultOutcomeID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FText DefaultPreviewReactionText;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TArray<FCommissionLeadDefinition> Leads;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TArray<FCommissionBranchRule> Branches;
    bool ValidateDefinition(FString& OutError) const;
    FCommissionRuleSetRow BuildRuleSet() const;
};
```

Build `FResolvedCommissionDefinition` from the asset without consulting `DT_Quests`, `DT_Documents`, or `CommissionRulesTable`. Store the definition pointer in drafts and instances so later evaluation/resolution uses the same authoritative object. Build `FNoticeQuest` directly from `Definition->Display`. Change `Story: Set Notice Quest` to expose only `Commission` plus slot/state.

- [ ] **Step 4: Run focused automation tests and verify GREEN**

Run: `Automation RunTests Dispatch.Commission.DataAsset; Dispatch.Story.Flow.StateTree.Api; Quit`.

Expected: all named tests pass with zero errors.

---

### Task 2: Dialogue DataAsset and Standard Effects

**Files:**
- Create: `Source/Dispatch/Public/Quest/DialogueDefinition.h`
- Create: `Source/Dispatch/Private/Quest/DialogueDefinition.cpp`
- Modify: `Source/Dispatch/Public/Quest/DispatchDialogueTypes.h`
- Modify: `Source/Dispatch/Public/Quest/DialogueSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/DialogueSubsystem.cpp`
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Test: `Source/Dispatch/Private/Tests/DialogueDefinitionTests.cpp`
- Test: `Source/Dispatch/Private/Tests/DialogueEffectTests.cpp`
- Test: `Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp`

**Interfaces:**
- Produces: `UDialogueDefinition`, `FDialogueNodeDefinition`, `FDialogueEffect`, `StartDialogue(UDialogueDefinition*)`, `StartDialogueForNPC(ANPCCharacter*, UDialogueDefinition*)`.
- Consumes: `UClueSubsystem`, `UStoryWorldStateSubsystem`, and commission APIs; Task 3 completes `UnlockLead` persistence.

- [ ] **Step 1: Write failing graph/effect tests**

```cpp
UDialogueDefinition* Dialogue = NewObject<UDialogueDefinition>();
Dialogue->DialogueID = TEXT("Dialogue.Test");
Dialogue->StartNodeID = TEXT("Start");
Dialogue->Nodes = {MakeLine(TEXT("Start"), TEXT("Exit")), MakeExit(TEXT("Exit"))};
TestTrue(TEXT("Reachable dialogue validates"), Dialogue->ValidateDefinition(Error));
TestTrue(TEXT("Subsystem starts direct asset"), System->StartDialogue(Dialogue));

FDialogueEffect Effect;
Effect.Type = EDialogueEffectType::SetNamedResult;
Effect.TargetID = TEXT("Dialogue.Result");
Effect.ValueID = TEXT("Accepted");
Dialogue->Nodes[0].Effects.Add(Effect);
TestEqual(TEXT("Entering node applies result"), WorldState->GetNamedResult(Effect.TargetID), Effect.ValueID);
```

- [ ] **Step 2: Build and verify RED**

Expected: compilation fails on missing dialogue asset/effect symbols.

- [ ] **Step 3: Implement asset-local node lookup, validation, and effects**

```cpp
UENUM(BlueprintType)
enum class EDialogueEffectType : uint8
{
    GrantClue, UnlockLead, SetStoryTag, SetNamedResult, OpenCommission, FinishDialogue
};

USTRUCT(BlueprintType)
struct FDialogueNodeDefinition
{
    GENERATED_BODY()
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FName NodeID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FDialogueNodeRow Node;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TArray<FDialogueEffect> Effects;
};
```

The subsystem holds `TObjectPtr<const UDialogueDefinition> ActiveDialogueDefinition`; `FindNodeRow` reads the active asset first. Apply a node's effects exactly once when it becomes current. Broken links close safely. Update `Story: Start Dialogue` to expose a `Dialogue` asset instead of `DialogueTreeID`.

- [ ] **Step 4: Run dialogue and StateTree tests and verify GREEN**

Run: `Automation RunTests Dispatch.Dialogue.Definition; Dispatch.Dialogue.Effects; Dispatch.NPC.Dialogue; Dispatch.Story.Flow.StateTree.Api; Quit`.

---

### Task 3: Clue Provenance and Two-Clue Leads

**Files:**
- Modify: `Source/Dispatch/Public/Quest/ClueSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/ClueSubsystem.cpp`
- Modify: `Source/Dispatch/Public/Commission/CommissionDefinition.h`
- Test: `Source/Dispatch/Private/Tests/ClueLeadTests.cpp`

**Interfaces:**
- Produces: `FCollectedClueRecord`, `FUnlockedLeadRecord`, `CollectClueWithContext`, `UnlockLead`, `HasLead`, `TryUnlockLeadFromPair`, `GetCollectedClueRecords`, `GetUnlockedLeadRecords`.
- Consumes: `FCommissionLeadDefinition` from Task 1.

- [ ] **Step 1: Write failing provenance/order-independent tests**

```cpp
TestTrue(TEXT("Context collection succeeds"), Clues->CollectClueWithContext(TEXT("Clue.A"), 3, TEXT("Letter.A"), TEXT("Commission.A")));
TestEqual(TEXT("Acquisition day retained"), Clues->GetCollectedClueRecords()[0].AcquiredDay, 3);
TestTrue(TEXT("Pair unlocks in reverse order"), Clues->TryUnlockLeadFromPair(Definition, TEXT("Clue.B"), TEXT("Clue.A"), 3, LeadID));
TestEqual(TEXT("Configured lead unlocked"), LeadID, FName(TEXT("Lead.AB")));
TestFalse(TEXT("Second unlock is rejected"), Clues->UnlockLead(TEXT("Lead.AB"), 3, Definition->CommissionID));
```

- [ ] **Step 2: Build and verify RED**

Expected: compilation fails on missing record and lead APIs.

- [ ] **Step 3: Implement records without duplicating clue definitions**

Store records keyed by ID; `HasClue`/`GetCollectedClueIDs` continue to read the new map. Pair matching compares `(A,B)` in both orders. Validate both clue definitions through `DT_Clues`; store only IDs, day, source, and associated commission.

- [ ] **Step 4: Run clue tests and verify GREEN**

Run: `Automation RunTests Dispatch.Clue; Quit`.

---

### Task 4: Daily Writing Types and Runtime Owner

**Files:**
- Create: `Source/Dispatch/Public/Writing/DailyWritingTypes.h`
- Create: `Source/Dispatch/Public/Writing/DailyWritingSubsystem.h`
- Create: `Source/Dispatch/Private/Writing/DailyWritingSubsystem.cpp`
- Modify: `Source/Dispatch/Public/Story/Rules/StoryRuleTypes.h`
- Modify: `Source/Dispatch/Public/Commission/CommissionSubsystem.h`
- Test: `Source/Dispatch/Private/Tests/DailyWritingSubsystemTests.cpp`

**Interfaces:**
- Produces: `FDailyWritingTemplate`, `FDailyWritingSection`, `FDailyWritingOption`, `FDailyWritingCondition`, `FSubmittedDailyRecord`, `UDailyWritingSubsystem::{BeginDay,GetAvailableOptions,SelectOption,SubmitCurrentDay,HasSubmittedDay}`.
- Consumes: clue/lead records, resolved commission instances, story tags/named results.

- [ ] **Step 1: Write failing validation/filter/submission tests**

```cpp
UDayDefinition* Day = NewObject<UDayDefinition>();
Day->DayNumber = 3;
Day->DailyWriting.Sections = MakeThreeSectionsWithDefaults();
TestTrue(TEXT("Valid day starts"), Writing->BeginDay(Day));
TestEqual(TEXT("Only default visible before clue"), Writing->GetAvailableOptions(TEXT("Event")).Num(), 1);
Clues->CollectClueWithContext(TEXT("Clue.A"), 3, TEXT("Test"), NAME_None);
TestEqual(TEXT("Clue option becomes visible"), Writing->GetAvailableOptions(TEXT("Event")).Num(), 2);
TestFalse(TEXT("Incomplete record cannot submit"), Writing->SubmitCurrentDay(Record));
SelectAllThreeSections(Writing);
TestTrue(TEXT("Complete record submits once"), Writing->SubmitCurrentDay(Record));
TestFalse(TEXT("Day cannot submit twice"), Writing->SubmitCurrentDay(Record));
TestEqual(TEXT("Credibility averages to expected grade"), Record.CredibilityGrade, EWritingCredibilityGrade::Plausible);
```

- [ ] **Step 2: Build and verify RED**

Expected: compilation fails on missing daily-writing types/subsystem and `UDayDefinition::DailyWriting`.

- [ ] **Step 3: Implement validation, visibility, credibility, and atomic submission**

```cpp
UENUM(BlueprintType)
enum class EWritingCredibilityGrade : uint8 { Unformed, Doubtful, Plausible, Strong };
UENUM(BlueprintType)
enum class EWritingStance : uint8 { Neutral, Truthful, Cautious, Concealing, Official };
```

`BeginDay` rejects templates that do not contain exactly three sections, duplicate IDs, duplicate option IDs, missing/multiple defaults, or conditional defaults. Visibility is default-or-all-conditions-met. Selection stores `OptionID`, not array index. Submission preflights every selection, computes a rounded average grade, builds stance counts/key tags, then atomically appends the record and updates `StoryWorldStateSubsystem` values. On failure it writes nothing.

- [ ] **Step 4: Run writing tests and verify GREEN**

Run: `Automation RunTests Dispatch.Writing; Quit`.

---

### Task 5: Day Flow, Writing Desk, and StateTree Integration

**Files:**
- Modify: `Source/Dispatch/Public/Story/Flow/DayFlowManager.h`
- Modify: `Source/Dispatch/Private/Story/Flow/DayFlowManager.cpp`
- Modify: `Source/Dispatch/Public/Items/WritingDesk.h`
- Modify: `Source/Dispatch/Private/Items/WritingDesk.cpp`
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Modify: `Config/DefaultGameplayTags.ini`
- Test: `Source/Dispatch/Private/Tests/DailyWritingFlowTests.cpp`

**Interfaces:**
- Produces: `ADayFlowManager::TryEndCurrentDay` gate on submitted writing, `AWritingDesk::SubmitDailyWriting`, `Story.Trigger.DailyWritingSubmitted`.
- Consumes: Task 4 submission event/record.

- [ ] **Step 1: Write failing end-of-day gate and signal tests**

```cpp
TestFalse(TEXT("Completed work alone cannot end configured writing day"), Flow->TryEndCurrentDay());
SubmitThreeSectionRecord(Writing);
TestTrue(TEXT("Submitted writing unlocks transition"), Flow->TryEndCurrentDay());
TestTrue(TEXT("Submission signal tag exists"), FGameplayTag::RequestGameplayTag(TEXT("Story.Trigger.DailyWritingSubmitted"), false).IsValid());
```

- [ ] **Step 2: Build/run and verify RED**

Expected: flow ends without submission or the signal tag is absent.

- [ ] **Step 3: Integrate day activation and submission**

On day start, `ADayFlowManager` calls `DailyWritingSubsystem->BeginDay(CurrentDefinition)`. `CanEndCurrentDay` requires work completion and, when `DailyWriting.Sections` is configured, `HasSubmittedDay(CurrentDay)`. `AWritingDesk::SubmitDailyWriting` submits the record, sends `Story.Trigger.DailyWritingSubmitted`, ends the writing session, and leaves sleep/transition to StateTree/DayFlow.

- [ ] **Step 4: Run flow tests and verify GREEN**

Run: `Automation RunTests Dispatch.Writing.Flow; Dispatch.Story.Flow; Quit`.

---

### Task 6: Explicit Migration Compatibility and Regression

**Files:**
- Modify: `Source/Dispatch/Public/Quest/QuestSubsystem.h`
- Modify: `Source/Dispatch/Public/Quest/DispatchContentSettings.h`
- Modify: `Source/Dispatch/Public/Quest/DialogueSubsystem.h`
- Modify: `Source/Dispatch/Public/Commission/CommissionSubsystem.h`
- Modify: `Source/Dispatch/Private/Tests/QuestPrototypeTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/CommissionDefinitionTests.cpp`

**Interfaces:**
- Produces: compile-time/editor deprecation warnings on legacy ID/DT writing, commission, and dialogue entry points.
- Preserves: existing runtime behavior until content assets are migrated; new StateTree nodes never expose both ID and asset inputs.

- [ ] **Step 1: Add reflection tests that distinguish authoritative and legacy APIs**

```cpp
TestNotNull(TEXT("Daily writing runtime owner exists"), UDailyWritingSubsystem::StaticClass());
TestNotNull(TEXT("Legacy Quest subsystem remains loadable during migration"), UQuestSubsystem::StaticClass());
TestTrue(TEXT("Legacy PublishQuestByID is deprecated"), PublishFunction->HasMetaData(TEXT("DeprecatedFunction")));
```

- [ ] **Step 2: Verify RED**

Expected: legacy methods do not yet carry migration metadata.

- [ ] **Step 3: Add deprecation metadata and migration messages**

Use messages that identify the replacement exactly, for example: `Use UDailyWritingSubsystem with UDayDefinition::DailyWriting.` Keep old config fields until actual content assets have been migrated and reference-audited; do not silently delete live project settings.

- [ ] **Step 4: Build and run the full project automation suite**

Run build, then: `Automation RunTests Dispatch; Quit`.

Expected: build exit code 0 and all `Dispatch` automation tests pass.

- [ ] **Step 5: Audit the final diff and content status**

Run:

```powershell
git status --short
git diff --check
git diff -- Source/Dispatch Config/DefaultGameplayTags.ini
```

Confirm no changes were made to the user's pre-existing `WBP_PlayerNotebook`, `LetterWidgetBase`, or letter tests, and report that existing `.uasset` data still needs editor-side reassignment to the new DA fields before old DT assets/config can be deleted.
