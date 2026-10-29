# Story UI Preview and Clue Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Let a designer choose formal Day/Clue/Inference assets in Story Workspace and launch the real notebook UI in one isolated `L_StoryUIPreview` PIE map without touching the formal save or Demo runtime state.

**Architecture:** An editor-only immutable preview request is validated before PIE. `FStoryUIPreviewCoordinator` starts in-process PIE with a map override, seeds only that PIE GameInstance through public runtime subsystem APIs, creates the real `UPlayerNotebookWidget`, and discards all request/runtime state on EndPIE. The notebook uses catalog inference data first and retains the legacy Commission Lead path as fallback.

**Tech Stack:** Unreal Engine 5.6 C++, PIE (`FRequestPlaySessionParams`), `FEditorDelegates`, UMG, Story Workspace Slate, Unreal Automation Tests.

**Spec:** `C:\Users\DULEOLK\.codex\visualizations\2026\08\06\019fd933-cca8-7a52-aeaa-ae3639ae03f7\2026-08-26-story-workspace-content-authoring-design.md`

## Global Constraints

- This plan starts only after the content-foundation and Story Workspace editor plans pass.
- The preview map is `/Game/Developers/StoryPreview/L_StoryUIPreview` and is not referenced by Campaign or StateTree.
- The preview request exists only in editor memory; it is not a DA, save object, runtime debug panel, or Widget class default.
- Preview seeds the actual PIE `UClueSubsystem` and `UDailyWritingSubsystem` and opens the actual configured notebook Widget class.
- Starting preview never mutates an editor-world subsystem, formal save, Campaign, Catalog, Day, Clue, or Inference asset.
- `bEnableSimpleClueTestOnConstruct` remains available but is marked legacy until the user accepts the new preview in the editor.
- Invalid formal assets block preview with actionable messages.
- EndPIE always clears coordinator state, even when seeding or Widget creation fails.

---

## File Structure

### Runtime integration

- `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h` / `Private/Widget/PlayerNotebookWidget.cpp` — active-day resolution and catalog inference presentation.
- `Source/Dispatch/Private/Tests/PlayerNotebookCluePageTests.cpp` — formal Definition-driven notebook tests alongside legacy tests.

### Editor preview model and coordinator

- `Source/DispatchEditor/Public/StoryWorkspace/StoryUIPreviewRequest.h` / `Private/...cpp` — immutable clue-preview input and validation.
- `Source/DispatchEditor/Public/StoryWorkspace/StoryUIPreviewCoordinator.h` / `Private/...cpp` — PIE lifecycle and seeding.
- `Source/DispatchEditor/Private/StoryWorkspace/StoryPreviewMapBootstrap.h/.cpp` — non-destructive map creation.
- `Source/DispatchEditor/Private/Tests/StoryUIPreviewRequestTests.cpp` — request validation.
- `Source/DispatchEditor/Private/Tests/StoryUIPreviewCoordinatorTests.cpp` — isolated PIE lifecycle.

### Story Workspace UI

- `Source/DispatchEditor/Private/StoryWorkspace/SStoryValidationPreview.h/.cpp` — formal Day/Clue selection, validation, start/stop controls.
- `Source/DispatchEditor/Private/StoryWorkspace/StoryWorkspaceEditor.cpp` — own one coordinator per open Campaign and forward status.
- `Source/DispatchEditor/Public/StoryWorkspace/StoryWorkspaceEditorSettings.h` — preview map and notebook Widget defaults.
- `Content/Developers/StoryPreview/L_StoryUIPreview.umap` — shared editor-only empty preview world.

---

### Task 1: Replace notebook test-only inference branching with formal inference support

**Files:**
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Modify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`
- Modify: `Source/Dispatch/Private/Tests/PlayerNotebookCluePageTests.cpp`

**Interfaces:**
- Consumes: `UDailyWritingSubsystem::GetActiveDayDefinition()` and `UClueSubsystem::TryUnlockInferenceFromPair(...)` from the foundation plan.
- Preserves: all existing public notebook methods and legacy `StartSimpleClueTest()`.

- [ ] **Step 1: Write failing formal-asset notebook tests**

Build a fixture with transient `UClueDefinition` A/B/D, `UInferenceDefinition` AB, `UStoryContentCatalog`, `UDayDefinition`, real `UClueSubsystem`, and real `UDailyWritingSubsystem`. Give the day a valid three-section writing template, call `BeginDay`, and collect A/B/D for that day.

Assert:

```cpp
TestTrue(TEXT("Notebook resolves the active day without class-default duplication"), Notebook->BuildCluePage(1));
Notebook->SelectClue(TEXT("Clue.B"));
Notebook->SelectClue(TEXT("Clue.A"));
TestEqual(TEXT("Formal inference succeeds"), Notebook->GetClueComparisonState(), EClueComparisonState::Succeeded);
TestEqual(TEXT("Inference body is displayed"), Notebook->GetClueInferenceText(), InferenceAB->InferenceText);
TestEqual(TEXT("Direction is displayed"), Notebook->GetClueInvestigationDirection(), InferenceAB->InvestigationDirection);
TestTrue(TEXT("Inference is recorded by the subsystem"), Clues->HasInference(TEXT("Inference.AB")));
```

Also assert D+A produces the existing failure text and that a legacy Commission Lead still works when no formal inference matches.

- [ ] **Step 2: Run the focused test and verify failure**

Run `Dispatch.Widget.PlayerNotebook.CluePage`. Expected: formal day resolution or inference comparison fails because the notebook only resolves `NotebookDayDefinitions` and Commission Leads.

- [ ] **Step 3: Resolve the active formal day first**

In `BuildCluePage`, after validating the day number, ask the writing subsystem for `GetActiveDayDefinition()`. Use it when its `DayNumber` matches, then fall back to `NotebookDayDefinitions`, then the legacy simple-test day. Keep one `DisplayedClueDayDefinition` path after resolution.

- [ ] **Step 4: Resolve formal inference before legacy Lead fallback**

In `CompareSelectedClues`, after obtaining `UClueSubsystem`, call:

```cpp
UInferenceDefinition* Inference = nullptr;
if (Clues->TryUnlockInferenceFromPair(
        FirstSelectedClueID,
        SecondSelectedClueID,
        DisplayedClueDayDefinition->DayNumber,
        Inference))
{
    ClueComparisonState = EClueComparisonState::Succeeded;
    ClueComparisonResultText = Inference->MarginNote;
    ClueInferenceText = Inference->InferenceText.IsEmpty()
        ? Inference->MarginNote
        : Inference->InferenceText;
    ClueInvestigationDirection = Inference->InvestigationDirection;
    ClearClueSelection();
    RefreshClueComparisonResult();
    ScheduleClueInferenceResult();
    return true;
}
```

Only after this block, run the unchanged `DisplayedClueDayDefinition->NotebookCommissions` legacy loop.

- [ ] **Step 5: Mark the simple test as legacy without removing it**

Change only editor metadata/display categories to `Clue|Notebook|Debug|Legacy` and add a deprecation message pointing designers to Story Workspace UI Preview. Do not remove methods, fields, or tests in this plan.

- [ ] **Step 6: Run notebook, clue, commission, and writing tests**

Expected: formal inference tests pass and every existing legacy test remains green.

- [ ] **Step 7: Commit**

```powershell
git add Source/Dispatch/Public/Widget/PlayerNotebookWidget.h Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp Source/Dispatch/Private/Tests/PlayerNotebookCluePageTests.cpp
git commit -m "feat: drive notebook inference from formal assets"
```

### Task 2: Add the immutable preview request and validation

**Files:**
- Create: `Source/DispatchEditor/Public/StoryWorkspace/StoryUIPreviewRequest.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/StoryUIPreviewRequest.cpp`
- Create: `Source/DispatchEditor/Private/Tests/StoryUIPreviewRequestTests.cpp`

**Interfaces:**
- Produces: `FStoryCluePreviewRequest` and `Validate(TArray<FText>&) const`.

- [ ] **Step 1: Write failing request-validation tests**

Cover exact rules:

```cpp
Request.Campaign = Campaign;
Request.Day = Day;
Request.CollectedClues = {ClueA, ClueB};
Request.NotebookWidgetClass = UPlayerNotebookWidget::StaticClass();
Request.PreviewMap = FSoftObjectPath(TEXT("/Game/Maps/L_MainMenu.L_MainMenu"));
TestTrue(TEXT("Complete request validates"), Request.Validate(Errors));

TestFalse(TEXT("Missing campaign fails"), MissingCampaign.Validate(Errors));
TestFalse(TEXT("Day outside catalog fails"), ForeignDay.Validate(Errors));
TestFalse(TEXT("Fewer than two clues fails"), OneClue.Validate(Errors));
TestFalse(TEXT("Duplicate clue fails"), DuplicateClue.Validate(Errors));
TestFalse(TEXT("Unregistered clue fails"), ForeignClue.Validate(Errors));
TestFalse(TEXT("Missing widget class fails"), MissingWidget.Validate(Errors));
TestFalse(TEXT("Missing preview map fails"), MissingMap.Validate(Errors));
```

The pure request test uses the already existing `L_MainMenu` only as a known loadable `UWorld`. Task 3 separately creates and verifies the required `L_StoryUIPreview`; production editor settings never point at `L_MainMenu`.

- [ ] **Step 2: Build and confirm missing-request failure**

Expected: compilation fails because `FStoryCluePreviewRequest` does not exist.

- [ ] **Step 3: Implement the request**

Use soft references so editor selections remain safe across reloads:

```cpp
struct FStoryCluePreviewRequest
{
    TSoftObjectPtr<UStoryCampaignDefinition> Campaign;
    TSoftObjectPtr<UDayDefinition> Day;
    TArray<TSoftObjectPtr<UClueDefinition>> CollectedClues;
    TSoftClassPtr<UPlayerNotebookWidget> NotebookWidgetClass;
    FSoftObjectPath PreviewMap;
    bool Validate(TArray<FText>& OutErrors) const;
};
```

Validation loads the Campaign and Catalog, requires the selected Day and every clue to be registered in that exact Catalog, rejects duplicate clue paths, requires at least two clues acquired on the selected day, verifies the Widget class derives from `UPlayerNotebookWidget`, verifies the map resolves to `UWorld`, and includes asset paths in every error.

- [ ] **Step 4: Run request tests**

Run `Dispatch.Editor.StoryWorkspace.PreviewRequest`. Expected: all cases pass.

- [ ] **Step 5: Commit**

```powershell
git add Source/DispatchEditor/Public/StoryWorkspace/StoryUIPreviewRequest.h Source/DispatchEditor/Private/StoryWorkspace/StoryUIPreviewRequest.cpp Source/DispatchEditor/Private/Tests/StoryUIPreviewRequestTests.cpp
git commit -m "feat: add validated story UI preview requests"
```

### Task 3: Create the shared preview map non-destructively

**Files:**
- Create: `Source/DispatchEditor/Private/StoryWorkspace/StoryPreviewMapBootstrap.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/StoryPreviewMapBootstrap.cpp`
- Modify: `Source/DispatchEditor/Private/DispatchEditorModule.cpp`
- Create via editor command: `Content/Developers/StoryPreview/L_StoryUIPreview.umap`

**Interfaces:**
- Produces: `FStoryPreviewMapBootstrap::EnsurePreviewMap(FText& OutError)`.
- Produces console command: `StoryWorkspace.CreatePreviewMap`.

- [ ] **Step 1: Write a map-bootstrap idempotency test**

Assert that `EnsurePreviewMap` creates a loadable `UWorld` at the exact path when missing, and a second call returns the existing world without replacing its package or changing its persistent level.

- [ ] **Step 2: Run the test and verify missing-bootstrap failure**

Expected: compilation fails because the bootstrap does not exist.

- [ ] **Step 3: Implement exact-path world creation**

First query/load `/Game/Developers/StoryPreview/L_StoryUIPreview.L_StoryUIPreview`. If found, return success. Otherwise create `UWorld` through `UWorldFactory` in package `/Game/Developers/StoryPreview/L_StoryUIPreview`, register it with Asset Registry, mark the package dirty, and save it with `UPackage::SavePackage`. Do not load, save, or replace the user's currently open editor map.

Register/unregister `StoryWorkspace.CreatePreviewMap` with module lifetime.

- [ ] **Step 4: Generate and verify the map**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=StoryWorkspace.CreatePreviewMap;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: `Content/Developers/StoryPreview/L_StoryUIPreview.umap` exists and resolves as `UWorld`.

- [ ] **Step 5: Commit**

```powershell
git add Source/DispatchEditor/Private/StoryWorkspace/StoryPreviewMapBootstrap.h Source/DispatchEditor/Private/StoryWorkspace/StoryPreviewMapBootstrap.cpp Source/DispatchEditor/Private/DispatchEditorModule.cpp Content/Developers/StoryPreview/L_StoryUIPreview.umap
git commit -m "feat: add shared Story UI preview map"
```

### Task 4: Implement isolated PIE preview coordination

**Files:**
- Create: `Source/DispatchEditor/Public/StoryWorkspace/StoryUIPreviewCoordinator.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/StoryUIPreviewCoordinator.cpp`
- Create: `Source/DispatchEditor/Private/Tests/StoryUIPreviewCoordinatorTests.cpp`
- Modify: `Source/DispatchEditor/DispatchEditor.Build.cs`

**Interfaces:**
- Consumes: validated `FStoryCluePreviewRequest`.
- Produces: `StartCluePreview`, `StopPreview`, `IsPreviewRunning`, `GetLastError`, and `OnPreviewStateChanged`.

- [ ] **Step 1: Write failing coordinator lifecycle tests**

Use latent editor automation commands to start a valid request, wait for PIE, inspect only the PIE GameInstance, then end PIE. Assert:

```cpp
TestTrue(TEXT("Preview starts"), Coordinator->StartCluePreview(Request));
TestTrue(TEXT("PIE clue A is collected"), PieClues->HasClue(TEXT("Clue.A")));
TestTrue(TEXT("PIE clue B is collected"), PieClues->HasClue(TEXT("Clue.B")));
TestEqual(TEXT("PIE writing day"), PieWriting->GetCurrentDay(), Day->DayNumber);
TestNotNull(TEXT("Real notebook is in the PIE viewport"), Coordinator->GetActiveNotebookForTesting());
TestFalse(TEXT("Editor/request assets were not modified"), Campaign->GetOutermost()->IsDirty());
```

After `RequestEndPlayMap`, assert `IsPreviewRunning()` is false, no active notebook remains, and a second preview can start cleanly.

- [ ] **Step 2: Run the coordinator test and verify missing-type failure**

Expected: compilation fails because the coordinator does not exist.

- [ ] **Step 3: Implement start and PIE request**

`StartCluePreview` rejects an active/requested PIE session, validates the request, copies it into `TOptional<FStoryCluePreviewRequest> PendingRequest`, binds `FEditorDelegates::PostPIEStarted` and `EndPIE`, then calls:

```cpp
FRequestPlaySessionParams Params;
Params.SessionDestination = EPlaySessionDestinationType::InProcess;
Params.WorldType = EPlaySessionWorldType::PlayInEditor;
Params.GlobalMapOverride = Request.PreviewMap.GetLongPackageName();
Params.bAllowOnlineSubsystem = false;
GUnrealEd->RequestPlaySession(Params);
```

Do not change `EditorStartupMap`, `GameDefaultMap`, the current editor world, or Campaign StateTree.

- [ ] **Step 4: Seed only the PIE GameInstance and open the real Widget**

In `PostPIEStarted`, locate the `FWorldContext` with `WorldType == EWorldType::PIE` and a GameInstance. Then:

1. load Campaign, Catalog, Day, clues, and Widget class from the copied request;
2. call `PieClues->SetContentCatalog(Catalog)`;
3. call `PieClues->CollectClueWithContext(ClueID, DayNumber, TEXT("Editor.UIPreview"), NAME_None)` for each selected clue;
4. call `PieWriting->BeginDay(Day)` and abort with an actionable error if it fails;
5. resolve the first local `APlayerController` in the PIE world;
6. call `CreateWidget<UPlayerNotebookWidget>(PlayerController, NotebookWidgetClass)` and `AddToViewport`;
7. retain only a weak pointer to the Widget and broadcast Running.

All failures call `RequestEndPlayMap()` after recording `LastError`.

- [ ] **Step 5: Implement stop and unconditional cleanup**

`StopPreview` calls `GUnrealEd->RequestEndPlayMap()` only for an active/requested preview. `EndPIE` clears the active Widget weak pointer, pending request, errors only when the run succeeded, delegate state, and running flags. The coordinator destructor removes any delegate handles even if PIE ended externally.

- [ ] **Step 6: Run lifecycle tests**

Run `Dispatch.Editor.StoryWorkspace.PreviewCoordinator`. Expected: start, seed, Widget, end, cleanup, and restart cases pass.

- [ ] **Step 7: Commit**

```powershell
git add Source/DispatchEditor/Public/StoryWorkspace/StoryUIPreviewCoordinator.h Source/DispatchEditor/Private/StoryWorkspace/StoryUIPreviewCoordinator.cpp Source/DispatchEditor/Private/Tests/StoryUIPreviewCoordinatorTests.cpp Source/DispatchEditor/DispatchEditor.Build.cs
git commit -m "feat: run Story UI previews in isolated PIE"
```

### Task 5: Connect preview controls to Story Workspace

**Files:**
- Modify: `Source/DispatchEditor/Private/StoryWorkspace/SStoryValidationPreview.h`
- Modify: `Source/DispatchEditor/Private/StoryWorkspace/SStoryValidationPreview.cpp`
- Modify: `Source/DispatchEditor/Public/StoryWorkspace/StoryWorkspaceEditor.h`
- Modify: `Source/DispatchEditor/Private/StoryWorkspace/StoryWorkspaceEditor.cpp`
- Modify: `Source/DispatchEditor/Public/StoryWorkspace/StoryWorkspaceEditorSettings.h`
- Modify: `Source/DispatchEditor/Private/Tests/StoryWorkspaceEditorTests.cpp`

**Interfaces:**
- Consumes: Campaign Catalog and `FStoryUIPreviewCoordinator`.
- Produces: formal clue preview configuration and start/stop UI.

- [ ] **Step 1: Add failing preview-panel state tests**

Construct the tab with a Campaign containing one day and three clues. Assert the Day picker lists only Catalog days, the clue checklist lists only Catalog clues, Start is disabled with fewer than two clues, enabled with a valid day/two clues/widget/map, and changes to Running/Stop while the coordinator reports an active preview.

- [ ] **Step 2: Run the focused editor test and confirm failure**

Expected: the current validation shell has no preview controls.

- [ ] **Step 3: Add editor settings**

Use per-project editor settings:

```cpp
UPROPERTY(Config, EditAnywhere, Category="UI Preview")
FSoftObjectPath PreviewMap = FSoftObjectPath(TEXT("/Game/Developers/StoryPreview/L_StoryUIPreview.L_StoryUIPreview"));

UPROPERTY(Config, EditAnywhere, Category="UI Preview")
TSoftClassPtr<UPlayerNotebookWidget> NotebookWidgetClass =
    TSoftClassPtr<UPlayerNotebookWidget>(FSoftObjectPath(TEXT("/Game/BluePrints/Widget/Menu/WBP_PlayerNotebook.WBP_PlayerNotebook_C")));
```

- [ ] **Step 4: Implement the clue-preview form**

The Validation & Preview tab contains:

- preview type fixed to `线索笔记` for this first implementation;
- Day asset combo sourced from `Catalog->Days`;
- scrollable clue checklist sourced from `Catalog->Clues`;
- read-only list of Catalog inferences that become possible from checked clues;
- validation area showing exact request errors;
- `启动隔离预览` button when stopped and `停止预览` when running;
- status text for Starting, Running, Stopping, and Failed.

Starting builds one `FStoryCluePreviewRequest` from the current selections/settings and calls the coordinator. No selected value is written back to Campaign or Catalog.

- [ ] **Step 5: Own coordinator lifetime in the toolkit**

Create one coordinator per `FStoryWorkspaceEditor`, pass it to the tab, and stop its preview when the toolkit closes. Closing a content asset editor while preview is running must end only the preview PIE session and release delegates.

- [ ] **Step 6: Run panel and integration tests**

Run `Dispatch.Editor.StoryWorkspace`, `Dispatch.Widget.PlayerNotebook.CluePage`, and the preview coordinator suite. Expected: pass.

- [ ] **Step 7: Perform manual editor acceptance**

1. Open `DA_StoryCampaign_Demo`.
2. Add or choose a formal Day, Clue A, Clue B, Clue D, and one A+B Inference in the Catalog.
3. Open Validation & Preview, select the Day and A/B/D.
4. Start preview; verify the real `WBP_PlayerNotebook` opens.
5. Verify A+B shows the inference and D+A shows the failure message.
6. Press J/close controls as applicable and end PIE.
7. Reopen the formal Demo and verify no preview state or collected clue was saved.

Expected: all seven observations succeed before considering the legacy simple-test flag removable.

- [ ] **Step 8: Run full verification and commit**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
git add Source/DispatchEditor Source/Dispatch/Public/Widget/PlayerNotebookWidget.h Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp Source/Dispatch/Private/Tests/PlayerNotebookCluePageTests.cpp Config
git commit -m "feat: add unified Story UI clue preview"
```

Expected: build succeeds, all Dispatch tests pass, manual acceptance passes, and no preview state survives EndPIE.
