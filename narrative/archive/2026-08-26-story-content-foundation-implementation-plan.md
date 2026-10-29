# Story Content Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the typed runtime content foundation that lets one Campaign reference a validated catalog of letters, dialogues, NPCs, clues, inferences, commissions, and days without breaking the current DataTable-driven game.

**Architecture:** Runtime content stays in the `Dispatch` module. A native `IStoryContentDefinition` contract supplies common identity and validation, `UStoryContentCatalog` is the explicit authority list, and new `UClueDefinition`, `UInferenceDefinition`, and `UNPCDefinition` assets coexist with legacy clue rows and commission leads through a catalog-first/fallback-second resolver.

**Tech Stack:** Unreal Engine 5.6 C++, `UPrimaryDataAsset`, native `UInterface`, `TSoftObjectPtr`, `IDataValidation`, Unreal Automation Tests.

**Spec:** `C:\Users\DULEOLK\.codex\visualizations\2026\08\06\019fd933-cca8-7a52-aeaa-ae3639ae03f7\2026-08-26-story-workspace-content-authoring-design.md`

## Global Constraints

- Do not delete or rewrite `DT_Clues`, existing `FCommissionLeadDefinition`, or existing `.uasset` files in this plan.
- Runtime never scans folders to infer formal content; only `UStoryContentCatalog` is authoritative.
- Content IDs are non-empty and globally unique inside one catalog.
- `UInferenceDefinition` owns clue-combination meaning and produces a stable Narrative Fact; it does not directly open another UI.
- `Items` remains a reserved category because the project currently has `FDispatchItemInfo`, not a formal item Definition asset.
- Shipping builds must not acquire an editor-module dependency.
- Every task begins with a failing focused test and ends with a focused test run plus a commit.

---

## File Structure

### New runtime files

- `Source/Dispatch/Public/Story/Content/StoryContentDefinition.h` — shared content type enum and native interface.
- `Source/Dispatch/Public/Story/Content/StoryContentCatalog.h` — authoritative typed soft-reference catalog.
- `Source/Dispatch/Private/Story/Content/StoryContentCatalog.cpp` — lookup, enumeration, and cross-reference validation.
- `Source/Dispatch/Public/Quest/ClueDefinition.h` / `Private/Quest/ClueDefinition.cpp` — atomic clue asset and legacy-row conversion.
- `Source/Dispatch/Public/Quest/InferenceDefinition.h` / `Private/Quest/InferenceDefinition.cpp` — unordered clue combination, inference presentation, and fact creation.
- `Source/Dispatch/Public/NPC/NPCDefinition.h` / `Private/NPC/NPCDefinition.cpp` — NPC identity/default presentation asset.
- `Source/Dispatch/Private/Tests/StoryContentDefinitionTests.cpp` — common identity behavior.
- `Source/Dispatch/Private/Tests/StoryContentCatalogTests.cpp` — catalog identity, registration, and reference validation.
- `Source/Dispatch/Private/Tests/ClueInferenceDefinitionTests.cpp` — clue/inference validation, pair matching, and fact generation.
- `Source/Dispatch/Private/Tests/NPCDefinitionTests.cpp` — NPC definition validation.

### Existing files modified

- `Source/Dispatch/Public/Quest/LetterDefinition.h` and `Private/Quest/LetterDefinition.cpp` — implement the common contract.
- `Source/Dispatch/Public/Quest/DialogueDefinition.h` and `Private/Quest/DialogueDefinition.cpp` — implement the common contract.
- `Source/Dispatch/Public/Commission/CommissionDefinition.h` and `Private/Commission/CommissionDefinition.cpp` — implement the common contract and enumerate result asset references.
- `Source/Dispatch/Public/Story/Rules/StoryRuleTypes.h` and `Private/Story/Rules/StoryRuleTypes.cpp` — make `UDayDefinition` expose identity/validation through the common contract.
- `Source/Dispatch/Public/Story/Narrative/StoryCampaignDefinition.h` and `Private/Story/Narrative/StoryCampaignDefinition.cpp` — add `ContentCatalog` and `RootStateTree` references and validate them.
- `Source/Dispatch/Public/Quest/ClueSubsystem.h` and `Private/Quest/ClueSubsystem.cpp` — catalog-first clue/inference resolution with legacy fallback.
- `Source/Dispatch/Public/Writing/DailyWritingSubsystem.h` — expose the active day read-only so the notebook does not need a duplicated day list for preview.
- `Source/Dispatch/Private/Tests/ClueSubsystemTests.cpp` and `ClueLeadTests.cpp` — compatibility and inference runtime coverage.

---

### Task 1: Add the shared content contract and adapt existing Definitions

**Files:**
- Create: `Source/Dispatch/Public/Story/Content/StoryContentDefinition.h`
- Create: `Source/Dispatch/Private/Tests/StoryContentDefinitionTests.cpp`
- Modify: `Source/Dispatch/Public/Quest/LetterDefinition.h`
- Modify: `Source/Dispatch/Private/Quest/LetterDefinition.cpp`
- Modify: `Source/Dispatch/Public/Quest/DialogueDefinition.h`
- Modify: `Source/Dispatch/Private/Quest/DialogueDefinition.cpp`
- Modify: `Source/Dispatch/Public/Commission/CommissionDefinition.h`
- Modify: `Source/Dispatch/Private/Commission/CommissionDefinition.cpp`
- Modify: `Source/Dispatch/Public/Story/Rules/StoryRuleTypes.h`
- Modify: `Source/Dispatch/Private/Story/Rules/StoryRuleTypes.cpp`

**Interfaces:**
- Produces: `EStoryContentType`, `UStoryContentDefinition`, and `IStoryContentDefinition`.
- Produces exact native methods: `GetStoryContentID()`, `GetStoryContentDisplayName()`, `GetStoryContentType()`, `GetStoryContentIcon()`, `GetDirectStoryContentReferences(...)`, and `ValidateStoryContent(...)`.
- Existing validation remains the source of truth; interface methods wrap `IsDefinitionValid()` or `ValidateDefinition(...)` instead of duplicating rules.

- [ ] **Step 1: Write the failing identity tests**

Add `Dispatch.Story.Content.DefinitionContract` tests that create one letter, dialogue, commission, and day in memory; assert `Cast<IStoryContentDefinition>` succeeds and returns these exact mappings:

```cpp
Letter->LetterID = TEXT("Letter.A");
Letter->Title = FText::FromString(TEXT("Letter title"));
TestEqual(TEXT("Letter ID"), Content->GetStoryContentID(), FName(TEXT("Letter.A")));
TestEqual(TEXT("Letter type"), Content->GetStoryContentType(), EStoryContentType::Letter);

Dialogue->DialogueID = TEXT("Dialogue.A");
TestEqual(TEXT("Dialogue display fallback"),
    Content->GetStoryContentDisplayName().ToString(), FString(TEXT("Dialogue.A")));

Commission->CommissionID = TEXT("Commission.A");
Commission->Display.QuestTitle = FText::FromString(TEXT("Commission title"));
TestEqual(TEXT("Commission display"),
    Content->GetStoryContentDisplayName().ToString(), FString(TEXT("Commission title")));

Day->DayNumber = 2;
Day->DayTitle = FText::FromString(TEXT("Second day"));
TestEqual(TEXT("Day ID"), Content->GetStoryContentID(), FName(TEXT("Day.2")));
```

- [ ] **Step 2: Run the focused test and verify it fails to compile**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: compilation fails because `StoryContentDefinition.h` and the interface methods do not exist.

- [ ] **Step 3: Implement the native contract**

Create the enum and interface with these declarations:

```cpp
UENUM(BlueprintType)
enum class EStoryContentType : uint8
{
    Letter,
    Dialogue,
    NPC,
    Clue,
    Inference,
    Commission,
    Day
};

UINTERFACE(MinimalAPI)
class UStoryContentDefinition : public UInterface
{
    GENERATED_BODY()
};

class DISPATCH_API IStoryContentDefinition
{
    GENERATED_BODY()
public:
    virtual FName GetStoryContentID() const = 0;
    virtual FText GetStoryContentDisplayName() const = 0;
    virtual EStoryContentType GetStoryContentType() const = 0;
    virtual TSoftObjectPtr<UTexture2D> GetStoryContentIcon() const { return nullptr; }
    virtual void GetDirectStoryContentReferences(TArray<TSoftObjectPtr<UObject>>& OutReferences) const
    {
        OutReferences.Reset();
    }
    virtual bool ValidateStoryContent(TArray<FText>& OutErrors) const = 0;
};
```

Implement exact identity mappings:

- Letter: `LetterID`, `Title`, `Letter`, `InventoryIcon`.
- Dialogue: `DialogueID`, `FText::FromName(DialogueID)`, `Dialogue`, no icon.
- Commission: `CommissionID`, `Display.QuestTitle` falling back to `FText::FromName(CommissionID)`, `Commission`, `Display.QuestListIcon`.
- Day: `FName(*FString::Printf(TEXT("Day.%d"), DayNumber))`, `DayTitle`, `Day`, `NotebookImage`.

Commission direct references include each non-null `ReturnDialogue` and `ResultLetter`. Day direct references include each non-null `NotebookCommissions` entry. Validation wrappers convert existing single-string errors to one `FText` entry; the day implementation factors its current `IsDataValid` checks into a reusable `ValidateDefinition(TArray<FText>&)` method.

- [ ] **Step 4: Build and run the focused tests**

Run the editor build, then:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Story.Content.DefinitionContract;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: all `Dispatch.Story.Content.DefinitionContract` tests pass.

- [ ] **Step 5: Commit**

```powershell
git add Source/Dispatch/Public/Story/Content Source/Dispatch/Public/Quest/LetterDefinition.h Source/Dispatch/Private/Quest/LetterDefinition.cpp Source/Dispatch/Public/Quest/DialogueDefinition.h Source/Dispatch/Private/Quest/DialogueDefinition.cpp Source/Dispatch/Public/Commission/CommissionDefinition.h Source/Dispatch/Private/Commission/CommissionDefinition.cpp Source/Dispatch/Public/Story/Rules/StoryRuleTypes.h Source/Dispatch/Private/Story/Rules/StoryRuleTypes.cpp Source/Dispatch/Private/Tests/StoryContentDefinitionTests.cpp
git commit -m "feat: add shared story content contract"
```

### Task 2: Add clue, inference, and NPC Data Assets

**Files:**
- Create: `Source/Dispatch/Public/Quest/ClueDefinition.h`
- Create: `Source/Dispatch/Private/Quest/ClueDefinition.cpp`
- Create: `Source/Dispatch/Public/Quest/InferenceDefinition.h`
- Create: `Source/Dispatch/Private/Quest/InferenceDefinition.cpp`
- Create: `Source/Dispatch/Public/NPC/NPCDefinition.h`
- Create: `Source/Dispatch/Private/NPC/NPCDefinition.cpp`
- Create: `Source/Dispatch/Private/Tests/ClueInferenceDefinitionTests.cpp`
- Create: `Source/Dispatch/Private/Tests/NPCDefinitionTests.cpp`

**Interfaces:**
- Consumes: `IStoryContentDefinition` from Task 1 and `FStoryFactRecord` from `NarrativeTypes.h`.
- Produces: `UClueDefinition::MakeLegacyRow()`.
- Produces: `UInferenceDefinition::MatchesClues(...)` and `MakeProducedFactRecord(int32)`.
- Produces: `UNPCDefinition::MakeInitialInfo()`.

- [ ] **Step 1: Write failing definition tests**

Cover these exact cases:

```cpp
UClueDefinition* A = NewObject<UClueDefinition>();
A->ClueID = TEXT("Clue.A");
A->Title = FText::FromString(TEXT("A"));
TestTrue(TEXT("Complete clue validates"), A->ValidateStoryContent(Errors));
TestEqual(TEXT("Legacy row keeps subject"), A->MakeLegacyRow().SubjectID, A->SubjectID);

UInferenceDefinition* AB = NewObject<UInferenceDefinition>();
AB->InferenceID = TEXT("Inference.AB");
AB->RequiredClues = {A, B};
TestTrue(TEXT("Pair matches forward"), AB->MatchesClues(TEXT("Clue.A"), TEXT("Clue.B")));
TestTrue(TEXT("Pair matches reverse"), AB->MatchesClues(TEXT("Clue.B"), TEXT("Clue.A")));
TestFalse(TEXT("Duplicate clue is rejected"), Duplicate->ValidateStoryContent(Errors));

const FStoryFactRecord Fact = AB->MakeProducedFactRecord(3);
TestEqual(TEXT("Fact type"), Fact.Key.Type, EStoryFactType::LeadUnlocked);
TestEqual(TEXT("Fact source"), Fact.Key.SourceAsset.Get(), static_cast<UObject*>(AB));
TestEqual(TEXT("Fact ID"), Fact.Key.SubID, AB->InferenceID);
TestTrue(TEXT("Fact is true"), Fact.Value.BoolValue);
TestEqual(TEXT("Fact day"), Fact.DayNumber, 3);
```

NPC tests require non-empty `NPCID` and display name, and verify `MakeInitialInfo()` preserves ID, name, description, body state, work state, and trust.

- [ ] **Step 2: Run the focused build and confirm missing-type failure**

Run the editor build. Expected: compilation fails on missing `UClueDefinition`, `UInferenceDefinition`, and `UNPCDefinition`.

- [ ] **Step 3: Implement `UClueDefinition`**

Use these fields:

```cpp
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Clue") FName ClueID = NAME_None;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Clue") FText Title;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Clue", meta=(MultiLine="true")) FText Description;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Clue") TSoftObjectPtr<UTexture2D> Icon;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Clue") TSoftObjectPtr<UTexture2D> EvidenceImage;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Clue") FName SubjectID = NAME_None;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Clue") FName CategoryID = NAME_None;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Clue|Source") FName DefaultSourceID = NAME_None;
```

Validation rejects an empty ID or empty title. `MakeLegacyRow()` copies title, description, subject, and category without introducing a second source of truth.

- [ ] **Step 4: Implement `UInferenceDefinition`**

Use an array for stable editor ordering and enforce set semantics in validation:

```cpp
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Inference") FName InferenceID = NAME_None;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Inference") TArray<TSoftObjectPtr<UClueDefinition>> RequiredClues;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Inference") FText Title;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Inference", meta=(MultiLine="true")) FText MarginNote;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Inference", meta=(MultiLine="true")) FText InferenceText;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Inference", meta=(MultiLine="true")) FText InvestigationDirection;
```

`MatchesClues` only succeeds for a two-clue inference, rejects equal IDs, and compares without order. Definition validation requires an ID, title, at least two non-null unique clues, and non-empty referenced clue IDs. Catalog validation in Task 3 separately requires every referenced clue to be registered. `MakeProducedFactRecord` creates a boolean `LeadUnlocked` fact whose source is `this`, `SubID` is `InferenceID`, and value is true.

- [ ] **Step 5: Implement `UNPCDefinition`**

Use `NPCID`, `DisplayName`, `Description`, `Portrait`, `InitialBodyState`, `InitialCurrentState`, `InitialTrustLevel`, `ContentTags`, and optional `TSoftClassPtr<ANPCCharacter> CharacterClass`. Validation rejects an empty ID or display name. `MakeInitialInfo()` returns a populated `FNPCInfo` and never mutates the asset.

- [ ] **Step 6: Build and run focused tests**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Story.Content.Definitions;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: clue, inference, and NPC definition tests pass.

- [ ] **Step 7: Commit**

```powershell
git add Source/Dispatch/Public/Quest/ClueDefinition.h Source/Dispatch/Private/Quest/ClueDefinition.cpp Source/Dispatch/Public/Quest/InferenceDefinition.h Source/Dispatch/Private/Quest/InferenceDefinition.cpp Source/Dispatch/Public/NPC/NPCDefinition.h Source/Dispatch/Private/NPC/NPCDefinition.cpp Source/Dispatch/Private/Tests/ClueInferenceDefinitionTests.cpp Source/Dispatch/Private/Tests/NPCDefinitionTests.cpp
git commit -m "feat: add clue inference and npc definitions"
```

### Task 3: Add the authoritative story content catalog

**Files:**
- Create: `Source/Dispatch/Public/Story/Content/StoryContentCatalog.h`
- Create: `Source/Dispatch/Private/Story/Content/StoryContentCatalog.cpp`
- Create: `Source/Dispatch/Private/Tests/StoryContentCatalogTests.cpp`

**Interfaces:**
- Consumes all Definition classes and `IStoryContentDefinition`.
- Produces: typed arrays `Letters`, `Dialogues`, `NPCs`, `Clues`, `Inferences`, `Commissions`, and `Days`.
- Produces: `GetAllContent`, `FindContentByID`, `ContainsContent`, and `ValidateDefinition`.

- [ ] **Step 1: Write failing catalog tests**

Test exact behavior:

```cpp
Catalog->Clues = {ClueA};
Catalog->Inferences = {InferenceAB};
TestTrue(TEXT("Registered clue is contained"), Catalog->ContainsContent(ClueA));
TestEqual(TEXT("ID lookup returns clue"), Catalog->FindContentByID(TEXT("Clue.A")), static_cast<UObject*>(ClueA));
TestFalse(TEXT("Duplicate IDs across categories fail"), DuplicateCatalog->ValidateDefinition(Errors));
TestFalse(TEXT("Null catalog entries fail"), NullCatalog->ValidateDefinition(Errors));
TestFalse(TEXT("Inference requiring an unregistered clue fails"), UnregisteredReferenceCatalog->ValidateDefinition(Errors));
```

- [ ] **Step 2: Run the focused build and verify missing-catalog failure**

Run the editor build. Expected: compilation fails because `UStoryContentCatalog` does not exist.

- [ ] **Step 3: Implement typed enumeration and lookup**

Declare typed `TArray<TSoftObjectPtr<...>>` properties for the seven supported categories. Do not declare an `Items` UObject array. Centralize array traversal in `GetAllContent(TArray<TSoftObjectPtr<UObject>>& OutContent) const`; every lookup and validation must call that method so category behavior cannot drift.

`FindContentByID` loads registered soft references synchronously, casts to `IStoryContentDefinition`, and returns the first exact ID match. `ContainsContent` compares `FSoftObjectPath`, not only loaded pointer identity.

- [ ] **Step 4: Implement catalog validation**

Validation emits asset-specific errors for:

- null/unloadable soft references;
- objects not implementing `IStoryContentDefinition`;
- empty IDs;
- duplicate IDs across any categories;
- child Definition validation errors;
- direct content references absent from the catalog.

Implement `IsDataValid(FDataValidationContext&)` by forwarding each `ValidateDefinition` error to the context.

- [ ] **Step 5: Build and run catalog tests**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Story.Content.Catalog;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: all catalog tests pass.

- [ ] **Step 6: Commit**

```powershell
git add Source/Dispatch/Public/Story/Content/StoryContentCatalog.h Source/Dispatch/Private/Story/Content/StoryContentCatalog.cpp Source/Dispatch/Private/Tests/StoryContentCatalogTests.cpp
git commit -m "feat: add authoritative story content catalog"
```

### Task 4: Make Campaign the single workspace entry

**Files:**
- Modify: `Source/Dispatch/Public/Story/Narrative/StoryCampaignDefinition.h`
- Modify: `Source/Dispatch/Private/Story/Narrative/StoryCampaignDefinition.cpp`
- Modify: `Source/Dispatch/Private/Tests/NarrativeDefinitionTests.cpp`

**Interfaces:**
- Consumes: `UStoryContentCatalog`.
- Produces: `ContentCatalog` and `RootStateTree` Campaign properties.

- [ ] **Step 1: Add failing Campaign reference tests**

Extend the valid Campaign fixture with a valid empty catalog and assert:

```cpp
TestTrue(TEXT("Legacy campaign without a catalog remains runtime-compatible"), MissingCatalog->ValidateDefinition(Errors));
TestFalse(TEXT("Campaign with an invalid catalog is invalid"), InvalidCatalogCampaign->ValidateDefinition(Errors));
TestTrue(TEXT("Root StateTree is optional during the editor-foundation phase"), ValidCampaign->ValidateDefinition(Errors));
```

Both references remain optional in runtime validation during this compatibility phase because existing Campaign fixtures and future authoring shells may not have them yet. Story Workspace reports a missing Catalog or Root StateTree as an editor warning; when a Catalog is assigned, runtime validation validates it strictly.

- [ ] **Step 2: Run the focused narrative definition test and confirm failure**

Run `Dispatch.Narrative.Definition`. Expected: compilation fails because Campaign does not expose the new properties.

- [ ] **Step 3: Add Campaign references and validation**

Add:

```cpp
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative|Content")
TObjectPtr<UStoryContentCatalog> ContentCatalog = nullptr;

UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative|Flow")
TSoftObjectPtr<UStateTree> RootStateTree;
```

Validation calls `ContentCatalog->ValidateDefinition` whenever a Catalog is assigned. A null Catalog and null `RootStateTree` remain runtime-compatible until the later StoryDirector bridge plan; the editor service owns the missing-reference warnings.

- [ ] **Step 4: Run narrative and content tests**

Run `Dispatch.Narrative` and `Dispatch.Story.Content`. Expected: both suites pass.

- [ ] **Step 5: Commit**

```powershell
git add Source/Dispatch/Public/Story/Narrative/StoryCampaignDefinition.h Source/Dispatch/Private/Story/Narrative/StoryCampaignDefinition.cpp Source/Dispatch/Private/Tests/NarrativeDefinitionTests.cpp
git commit -m "feat: connect campaigns to story content catalogs"
```

### Task 5: Add catalog-first clue and inference runtime resolution

**Files:**
- Modify: `Source/Dispatch/Public/Quest/ClueSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/ClueSubsystem.cpp`
- Modify: `Source/Dispatch/Public/Writing/DailyWritingSubsystem.h`
- Modify: `Source/Dispatch/Private/Tests/ClueSubsystemTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/ClueLeadTests.cpp`

**Interfaces:**
- Consumes: `UStoryContentCatalog`, `UClueDefinition`, and `UInferenceDefinition`.
- Produces: `SetContentCatalog`, `FindClueDefinitionAsset`, `FindInferenceForPair`, `TryUnlockInferenceFromPair`, `HasInference`, and `GetUnlockedInferenceIDs`.
- Preserves: `FindClueDefinition(FName, FClueDefinitionRow&)` and `TryUnlockLeadFromPair(...)` for existing callers.
- Produces: `UDailyWritingSubsystem::GetActiveDayDefinition() const`.

- [ ] **Step 1: Write failing catalog-first and fallback tests**

Add tests for:

```cpp
Clues->SetContentCatalog(Catalog);
TestTrue(TEXT("Catalog clue can be collected"), Clues->CollectClueWithContext(TEXT("Clue.A"), 1, TEXT("Preview"), NAME_None));
TestTrue(TEXT("Catalog definition wins over a same-ID legacy row"), Clues->FindClueDefinition(TEXT("Clue.A"), Row));
TestEqual(TEXT("Catalog title wins"), Row.Title.ToString(), FString(TEXT("Catalog A")));

UInferenceDefinition* Unlocked = nullptr;
TestTrue(TEXT("Reverse pair unlocks inference"), Clues->TryUnlockInferenceFromPair(TEXT("Clue.B"), TEXT("Clue.A"), 1, Unlocked));
TestEqual(TEXT("Unlocked definition"), Unlocked, InferenceAB);
TestTrue(TEXT("Inference is queryable"), Clues->HasInference(TEXT("Inference.AB")));

LegacyOnlyClues->SetClueTableForTesting(LegacyTable);
TestTrue(TEXT("Legacy table remains a fallback"), LegacyOnlyClues->CollectClue(TEXT("Clue.Legacy")));
```

- [ ] **Step 2: Run `Dispatch.Clue` and confirm new API failure**

Expected: compilation fails because the catalog and inference APIs do not exist.

- [ ] **Step 3: Implement catalog-first resolution**

Store a transient `TObjectPtr<UStoryContentCatalog> ContentCatalog`. `FindClueDefinitionAsset` uses `ContentCatalog->FindContentByID`, casts to `UClueDefinition`, and never scans folders. Existing `FindClueDefinition` first converts a catalog asset through `MakeLegacyRow()`, then falls back to `ResolveClueTable()`.

Store unlocked inference IDs in a transient `TMap<FName, FUnlockedInferenceRecord>` where the record contains `InferenceID`, `UnlockedDay`, and `FStoryFactRecord ProducedFact`. `TryUnlockInferenceFromPair` requires both clues to be collected, uses `FindInferenceForPair`, is idempotent, and fills `ProducedFact` through `MakeProducedFactRecord` without directly opening any other system.

- [ ] **Step 4: Expose the active day definition**

Add:

```cpp
UFUNCTION(BlueprintPure, Category="Writing")
UDayDefinition* GetActiveDayDefinition() const { return ActiveDayDefinition; }
```

This is read-only and lets the notebook resolve the same day object already owned by the real daily-writing subsystem.

- [ ] **Step 5: Run compatibility tests**

Run `Dispatch.Clue`, `Dispatch.Commission`, `Dispatch.Writing`, and `Dispatch.Widget.PlayerNotebook.CluePage`. Expected: new inference tests and all legacy lead/DataTable tests pass.

- [ ] **Step 6: Run full build and regression**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: build succeeds and the complete Dispatch suite has zero failures.

- [ ] **Step 7: Commit**

```powershell
git add Source/Dispatch/Public/Quest/ClueSubsystem.h Source/Dispatch/Private/Quest/ClueSubsystem.cpp Source/Dispatch/Public/Writing/DailyWritingSubsystem.h Source/Dispatch/Private/Tests/ClueSubsystemTests.cpp Source/Dispatch/Private/Tests/ClueLeadTests.cpp
git commit -m "feat: resolve clues and inferences from story catalogs"
```
