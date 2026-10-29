# Lightweight NPC Story Scene System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a data-driven NPC scene runtime that claims an NPC, resolves a level anchor, arrives by keep/teleport/move, applies a lightweight routine, plays an NPC-scoped entry dialogue, and preserves a scene-specific interaction dialogue.

**Architecture:** A `UWorldSubsystem` owns one state-machine record per controlled NPC. Level anchor actors register by `AnchorID`; the dialogue subsystem carries weak NPC context; the routine component supplies state/movement events and anchor-bounded wandering; one StateTree task starts and polls a scene while leaving successful scenes active.

**Tech Stack:** Unreal Engine 5.6 C++, `UWorldSubsystem`, `UGameInstanceSubsystem`, `UDataTable`, `AAIController`, NavigationSystem, StateTree, Unreal Automation Tests.

## Global Constraints

- Preserve all current NPC Blueprint dialogue/UI events and the existing `Story: Start Dialogue`, `Story: Set NPC Behavior`, and `Story: Wait For Dialogue End` nodes.
- Preserve `StoryDialogueTreeID`, `DailyDialogueTreeID`, `ENPCRoutineState`, and `UDialogueSubsystem`; extend them instead of replacing them.
- Never store world coordinates in `FNPCSceneRow`.
- Do not add a full schedule system, day/night routine, Behavior Tree, or the report-writing runtime feature.
- Invalid scene, NPC, anchor, dialogue, navigation, or movement input fails safely with an ID-rich log; movement failure never silently succeeds.
- Do not create or edit `.uasset` content in this implementation.
- Use test-first red/green cycles for every production behavior.

---

## File Map

- Create `Source/Dispatch/Public/NPC/NPCSceneTypes.h`: scene enums, DataTable row, runtime status.
- Create `Source/Dispatch/Public/NPC/NPCSceneAnchor.h` and `Source/Dispatch/Private/NPC/NPCSceneAnchor.cpp`: level anchor registration.
- Create `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h` and `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp`: scene lookup, ownership, arrival, dialogue, cleanup, and status.
- Modify `Source/Dispatch/Public/Quest/DispatchContentSettings.h` and `Source/Dispatch/Private/Quest/DispatchContentSettings.cpp`: NPC scene table setting.
- Modify `Source/Dispatch/Public/Quest/DialogueSubsystem.h` and `Source/Dispatch/Private/Quest/DialogueSubsystem.cpp`: NPC-scoped dialogue context.
- Modify `Source/Dispatch/Public/Component/NPCDialogueComponent.h` and `Source/Dispatch/Private/Component/NPCDialogueComponent.cpp`: transient scene interaction override.
- Modify `Source/Dispatch/Public/Component/NPCRoutineComponent.h` and `Source/Dispatch/Private/Component/NPCRoutineComponent.cpp`: state/move delegates and bounded wandering.
- Modify `Source/Dispatch/Private/Characters/NPCCharacter.cpp`: prefer a configured dialogue tree on interaction without removing Blueprint fallback.
- Modify `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h` and `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`: `Story: Play NPC Scene` and NPC-aware legacy start node.
- Modify `Source/Dispatch/Dispatch.Build.cs`: add the `NavigationSystem` dependency.
- Create `Source/Dispatch/Private/Tests/NPCSceneTests.cpp`: scene data, lookup, arrival, lifecycle, and dialogue orchestration tests.
- Modify `Source/Dispatch/Private/Tests/NPCDialogueTests.cpp`: NPC-context dialogue tests.
- Modify `Source/Dispatch/Private/Tests/NPCSystemTests.cpp`: routine delegates and interaction fallback tests.
- Modify `Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp`: StateTree reflected API and status mapping tests.

---

### Task 1: Scene Data Contract and Project Settings

**Files:**
- Create: `Source/Dispatch/Public/NPC/NPCSceneTypes.h`
- Modify: `Source/Dispatch/Public/Quest/DispatchContentSettings.h`
- Modify: `Source/Dispatch/Private/Quest/DispatchContentSettings.cpp`
- Create: `Source/Dispatch/Private/Tests/NPCSceneTests.cpp`

**Interfaces:**
- Produces: `ENPCSceneArrivalMode`, `ENPCSceneExitPolicy`, `ENPCSceneStatus`, `FNPCSceneRow`.
- Produces: `const UDataTable* UDispatchContentSettings::LoadNPCSceneTable() const`.

- [ ] **Step 1: Write the failing reflected-structure test**

Add `FNPCSceneTypesTest` under `Dispatch.NPC.Scene.Types`. The assertions must name every field and verify the row base:

```cpp
TestTrue(TEXT("Scene row is a table row"),
    FNPCSceneRow::StaticStruct()->IsChildOf(FTableRowBase::StaticStruct()));
TestNotNull(TEXT("NPCID exists"), FindFProperty<FNameProperty>(FNPCSceneRow::StaticStruct(), TEXT("NPCID")));
TestNotNull(TEXT("AnchorID exists"), FindFProperty<FNameProperty>(FNPCSceneRow::StaticStruct(), TEXT("AnchorID")));
TestNotNull(TEXT("ArrivalMode exists"), FindFProperty<FEnumProperty>(FNPCSceneRow::StaticStruct(), TEXT("ArrivalMode")));
TestNotNull(TEXT("AcceptanceRadius exists"), FindFProperty<FFloatProperty>(FNPCSceneRow::StaticStruct(), TEXT("AcceptanceRadius")));
TestNotNull(TEXT("RoutineState exists"), FindFProperty<FEnumProperty>(FNPCSceneRow::StaticStruct(), TEXT("RoutineState")));
TestNotNull(TEXT("Entry dialogue exists"), FindFProperty<FNameProperty>(FNPCSceneRow::StaticStruct(), TEXT("EntryDialogueTreeID")));
TestNotNull(TEXT("Interaction dialogue exists"), FindFProperty<FNameProperty>(FNPCSceneRow::StaticStruct(), TEXT("InteractionDialogueTreeID")));
TestNotNull(TEXT("Wait flag exists"), FindFProperty<FBoolProperty>(FNPCSceneRow::StaticStruct(), TEXT("bWaitForEntryDialogue")));
TestNotNull(TEXT("Idle radius exists"), FindFProperty<FFloatProperty>(FNPCSceneRow::StaticStruct(), TEXT("IdleRadius")));
TestNotNull(TEXT("Exit policy exists"), FindFProperty<FEnumProperty>(FNPCSceneRow::StaticStruct(), TEXT("ExitPolicy")));
TestNotNull(TEXT("Settings expose NPCSceneTable"),
    FindFProperty<FObjectPropertyBase>(UDispatchContentSettings::StaticClass(), TEXT("NPCSceneTable")));
```

- [ ] **Step 2: Build to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: compilation fails because `NPC/NPCSceneTypes.h` or its declared types/settings property do not exist.

- [ ] **Step 3: Add the minimal types and loader**

Define the row exactly as approved:

```cpp
USTRUCT(BlueprintType)
struct DISPATCH_API FNPCSceneRow : public FTableRowBase
{
    GENERATED_BODY()
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene") FName NPCID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene") FName AnchorID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene") ENPCSceneArrivalMode ArrivalMode = ENPCSceneArrivalMode::KeepPosition;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene", meta=(ClampMin="0.0")) float AcceptanceRadius = 60.0f;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene") ENPCRoutineState RoutineState = ENPCRoutineState::Idle;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene") FName EntryDialogueTreeID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene") FName InteractionDialogueTreeID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene") bool bWaitForEntryDialogue = true;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene", meta=(ClampMin="0.0")) float IdleRadius = 150.0f;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="NPC|Scene") ENPCSceneExitPolicy ExitPolicy = ENPCSceneExitPolicy::StayAtAnchor;
};
```

Add `TSoftObjectPtr<UDataTable> NPCSceneTable` under `Content|NPC Scene` and return `NPCSceneTable.LoadSynchronous()` from `LoadNPCSceneTable()`.

- [ ] **Step 4: Build and run the focused test to verify GREEN**

Run the build command, then:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.NPC.Scene.Types;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: build exits `0`; test log reports `Dispatch.NPC.Scene.Types` successful.

- [ ] **Step 5: Commit only Task 1 files**

```powershell
git add Source/Dispatch/Public/NPC/NPCSceneTypes.h Source/Dispatch/Public/Quest/DispatchContentSettings.h Source/Dispatch/Private/Quest/DispatchContentSettings.cpp Source/Dispatch/Private/Tests/NPCSceneTests.cpp
git commit -m "feat: define NPC scene data"
```

### Task 2: NPC-Scoped Dialogue and Interaction Override

**Files:**
- Modify: `Source/Dispatch/Public/Quest/DialogueSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/DialogueSubsystem.cpp`
- Modify: `Source/Dispatch/Public/Component/NPCDialogueComponent.h`
- Modify: `Source/Dispatch/Private/Component/NPCDialogueComponent.cpp`
- Modify: `Source/Dispatch/Private/Characters/NPCCharacter.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCDialogueTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCSystemTests.cpp`

**Interfaces:**
- Produces: `bool StartDialogueForNPC(ANPCCharacter*, FName)` and `ANPCCharacter* GetCurrentDialogueNPC() const`.
- Produces: `FOnDialogueStartedForNPCNative::Broadcast(FName, ANPCCharacter*)` and `FOnDialogueEndedForNPCNative::Broadcast(FName, ANPCCharacter*)` while keeping existing delegates unchanged.
- Produces: `SetSceneInteractionDialogueTreeID`, `ClearSceneInteractionDialogueTreeID`, and `GetInteractionDialogueTreeID` on `UNPCDialogueComponent`.

- [ ] **Step 1: Write failing dialogue-context tests**

Add tests under `Dispatch.Dialogue.NPCContext` that create an in-memory `FDialogueNodeRow` table and two NPCs:

```cpp
TestTrue(TEXT("NPC dialogue starts"), System->StartDialogueForNPC(NPCA, TEXT("Tree.A")));
TestEqual(TEXT("Active NPC is retained"), System->GetCurrentDialogueNPC(), NPCA);
System->EndDialogue();
TestNull(TEXT("NPC context clears after end"), System->GetCurrentDialogueNPC());
TestFalse(TEXT("Null NPC is rejected"), System->StartDialogueForNPC(nullptr, TEXT("Tree.A")));
TestFalse(TEXT("Missing tree is rejected"), System->StartDialogueForNPC(NPCB, TEXT("Tree.Missing")));
TestNull(TEXT("Failed start leaves no attempted NPC"), System->GetCurrentDialogueNPC());
```

Bind the new native delegates and assert start/end both report the exact tree and NPC while the context remains readable. Add a replacement assertion: starting `Tree.B` for `NPCB` first ends `Tree.A` with `NPCA`.

Add component/character tests asserting override priority and fallback:

```cpp
Dialogue->StoryDialogueTreeID = TEXT("Story.Default");
Dialogue->DailyDialogueTreeID = TEXT("Daily.Default");
TestEqual(TEXT("Story is first default"), Dialogue->GetInteractionDialogueTreeID(), FName(TEXT("Story.Default")));
Dialogue->SetSceneInteractionDialogueTreeID(TEXT("Scene.Short"));
TestEqual(TEXT("Scene override wins"), Dialogue->GetInteractionDialogueTreeID(), FName(TEXT("Scene.Short")));
Dialogue->ClearSceneInteractionDialogueTreeID();
TestEqual(TEXT("Clearing restores story"), Dialogue->GetInteractionDialogueTreeID(), FName(TEXT("Story.Default")));
```

- [ ] **Step 2: Build to verify RED**

Run the Task 1 build command. Expected: missing NPC-context and interaction-override methods/delegate.

- [ ] **Step 3: Implement dialogue context without changing legacy delegate signatures**

Add a weak current NPC, two two-parameter native delegates, and a behavior/routine snapshot for the current NPC. Refactor `StartDialogue` through one private method so table/row validation happens before an active dialogue is ended:

```cpp
bool UDialogueSubsystem::StartDialogueForNPC(ANPCCharacter* TargetNPC, FName DialogueTreeID)
{
    if (!IsValid(TargetNPC)) return false;
    return StartDialogueInternal(DialogueTreeID, TargetNPC);
}
```

After validation succeeds, end any prior dialogue, cache the target NPC's behavior/routine, set `InConversation`/`Talking`, assign IDs/context, and broadcast the NPC-aware start delegate. In `EndDialogue`, capture `EndedID` and `EndedNPC`, clear IDs, restore the captured behavior/routine, broadcast existing dynamic/native delegates, broadcast `OnDialogueEndedForNPCNative(EndedID, EndedNPC)`, then clear `CurrentDialogueNPC`. Do not alter `FOnDialogueEnded` or `OnDialogueEndedNative`.

Implement component resolution as scene override, story default, then daily default. In `ANPCCharacter::InteractWithPlayer`, call `StartDialogueForNPC(this, ResolvedID)` when a non-empty tree resolves; if it returns false or no tree resolves, execute the current `EnterConversation` Blueprint/manual path unchanged. Because the dialogue subsystem owns the snapshot, a successful tree dialogue restores the exact pre-dialogue scene routine instead of forcing `Idle`.

- [ ] **Step 4: Build and run focused dialogue/NPC tests**

Run the build, then run `Automation RunTests Dispatch.Dialogue.NPCContext;Quit` and `Automation RunTests Dispatch.NPC.Character;Quit`. Expected: both commands report success and existing UI interaction tests stay green.

- [ ] **Step 5: Commit Task 2 files**

```powershell
git add Source/Dispatch/Public/Quest/DialogueSubsystem.h Source/Dispatch/Private/Quest/DialogueSubsystem.cpp Source/Dispatch/Public/Component/NPCDialogueComponent.h Source/Dispatch/Private/Component/NPCDialogueComponent.cpp Source/Dispatch/Private/Characters/NPCCharacter.cpp Source/Dispatch/Private/Tests/NPCDialogueTests.cpp Source/Dispatch/Private/Tests/NPCSystemTests.cpp
git commit -m "feat: add NPC-scoped dialogue context"
```

### Task 3: Anchor Registry and Scene Validation

**Files:**
- Create: `Source/Dispatch/Public/NPC/NPCSceneAnchor.h`
- Create: `Source/Dispatch/Private/NPC/NPCSceneAnchor.cpp`
- Create: `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h`
- Create: `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCSceneTests.cpp`

**Interfaces:**
- Produces: the five Blueprint APIs from the design plus `AbortNPCScene(FName NPCID)` and `GetNPCIDForScene(FName SceneID) const` for StateTree cleanup.
- Produces: `RegisterAnchor(ANPCSceneAnchor*)` and `UnregisterAnchor(ANPCSceneAnchor*)`.
- Produces test hooks: `SetNPCSceneTableForTesting(UDataTable*)` and direct actor lookup through the test world.

- [ ] **Step 1: Write failing validation tests**

Create a test map, an in-memory scene table, and helper `AddSceneRow`. Test these independent cases under `Dispatch.NPC.Scene.Validation`: invalid scene ID, row with empty `NPCID`, missing NPC, duplicate NPCs with the same ID, empty required anchor, missing anchor, and two registered anchors with one ID. Each call must return false and leave no active scene.

```cpp
TestFalse(TEXT("Unknown scene fails"), Scenes->PlayNPCScene(TEXT("Scene.Missing")));
TestFalse(TEXT("Missing NPC fails"), Scenes->PlayNPCScene(TEXT("Scene.NoNPC")));
TestFalse(TEXT("Missing anchor fails"), Scenes->PlayNPCScene(TEXT("Scene.NoAnchor")));
TestFalse(TEXT("Duplicate anchor is ambiguous"), Scenes->PlayNPCScene(TEXT("Scene.DuplicateAnchor")));
```

Destroy the only valid anchor, tick/purge the world, and verify lookup fails instead of dereferencing the weak pointer.

- [ ] **Step 2: Build to verify RED**

Run the editor build. Expected: missing anchor and scene subsystem types/APIs.

- [ ] **Step 3: Implement the registry and validation-only scene start**

Define `ANPCSceneAnchor` with `AnchorID` and `IdleRadiusOverride`; call the world subsystem from `BeginPlay`/`EndPlay`. In the subsystem, store `TMap<FName, TArray<TWeakObjectPtr<ANPCSceneAnchor>>>`, compact invalid pointers before each lookup, and require exactly one live match.

Implement table resolution and row validation. Scan `TActorIterator<ANPCCharacter>` for exactly one `NPCInfo.NPCID` match. For now, after successful validation/cache, create an active record with `ENPCSceneStatus::Active`; later tasks replace that terminal shortcut with arrival/dialogue phases. Keep asynchronously failed records with status `Failed` long enough for StateTree to observe them, while `IsNPCSceneActive` returns false for `Failed`; `StopNPCScene` or a replacement removes the terminal record.

- [ ] **Step 4: Build and run validation tests**

Run build and `Automation RunTests Dispatch.NPC.Scene.Validation;Quit`. Expected: all validation cases pass without access violations.

- [ ] **Step 5: Commit Task 3 files**

```powershell
git add Source/Dispatch/Public/NPC/NPCSceneAnchor.h Source/Dispatch/Private/NPC/NPCSceneAnchor.cpp Source/Dispatch/Public/NPC/NPCSceneSubsystem.h Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp Source/Dispatch/Private/Tests/NPCSceneTests.cpp
git commit -m "feat: resolve NPC scene actors and anchors"
```

### Task 4: Teleport and Exit Policies

**Files:**
- Modify: `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h`
- Modify: `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCSceneTests.cpp`

**Interfaces:**
- Consumes: validated `FNPCSceneRow`, unique NPC and anchor.
- Produces: synchronous `KeepPosition`/`Teleport` arrival and reversible `FActiveNPCScene` snapshot.

- [ ] **Step 1: Write failing teleport and stop-policy tests**

Test `Teleport` copies anchor location and rotation and sets `StoryLocked`, configured routine, and interaction dialogue. Cache an original transform/behavior/routine/override before play.

For `RestorePrevious`, call `StopNPCScene(NPCID, true)` and assert all cached values are restored. For `StayAtAnchor`, call `StopNPCScene(NPCID, false)` and assert the anchor transform, scene routine, and scene interaction dialogue remain. Add a `KeepPosition` case asserting location is unchanged while routine/dialogue are applied.

- [ ] **Step 2: Run the focused test to verify RED**

Build and run `Automation RunTests Dispatch.NPC.Scene.Arrival;Quit`. Expected: assertions fail because Task 3 does not apply transforms or restoration.

- [ ] **Step 3: Implement snapshot, synchronous arrival, and cleanup**

Populate an `FActiveNPCScene` record before mutating the NPC:

```cpp
Record.PreviousTransform = NPC->GetActorTransform();
Record.PreviousBehaviorMode = NPC->BehaviorMode;
Record.PreviousRoutineState = NPC->NPCRoutineComponent->GetRoutineState();
Record.PreviousInteractionDialogueTreeID = NPC->NPCDialogueComponent->GetSceneInteractionDialogueTreeID();
```

Stop current AI movement, set `StoryLocked`, apply `KeepPosition` or `SetActorLocationAndRotation`, then apply configured routine/dialogue. Centralize rollback in `RestorePreviousState(FActiveNPCScene&)` and retained cleanup in `LeaveAtSceneState(FActiveNPCScene&)`.

- [ ] **Step 4: Build and verify arrival tests GREEN**

Run build and `Automation RunTests Dispatch.NPC.Scene.Arrival;Quit`. Expected: teleport, keep-position, restore, and stay tests pass.

- [ ] **Step 5: Commit Task 4 files**

```powershell
git add Source/Dispatch/Public/NPC/NPCSceneSubsystem.h Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp Source/Dispatch/Private/Tests/NPCSceneTests.cpp
git commit -m "feat: apply and restore NPC scene placement"
```

### Task 5: Routine Events, Directed Move, and Bounded Wandering

**Files:**
- Modify: `Source/Dispatch/Public/Component/NPCRoutineComponent.h`
- Modify: `Source/Dispatch/Private/Component/NPCRoutineComponent.cpp`
- Modify: `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h`
- Modify: `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp`
- Modify: `Source/Dispatch/Dispatch.Build.cs`
- Modify: `Source/Dispatch/Private/Tests/NPCSystemTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCSceneTests.cpp`

**Interfaces:**
- Produces dynamic multicast delegates with signatures `StateChanged(ENPCRoutineState, ENPCRoutineState)` and parameterless `MoveStarted()`, `MoveCompleted()`, and `MoveFailed()`.
- Produces `ConfigureRoutineArea(FVector Center, float Radius)`, `SetRoutinePaused(bool)`, `StartWandering()`, and `StopRoutineMovement()`.
- Produces scene status `Moving -> Active/Failed` from the AI move completion callback.

- [ ] **Step 1: Write failing routine-state and movement tests**

Bind native test lambdas to the delegates. Assert setting the same state emits zero changes and a real change emits exactly one old/new pair. For scene move, use a possessed test NPC and deterministic test hook that injects move completion; assert start changes routine to `GoingToPoint`, success applies the target routine, and failure restores the snapshot with status `Failed`.

For wandering, inject a navigation-point provider in test builds, configure center `(100, 200, 0)` and radius `150`, and assert every requested point is within that radius. Set `SetRoutinePaused(true)` and assert no further request occurs.

- [ ] **Step 2: Run tests to verify RED**

Build and run `Automation RunTests Dispatch.NPC.Routine;Quit` and `Automation RunTests Dispatch.NPC.Scene.Move;Quit`. Expected: missing delegates/methods and move status behavior.

- [ ] **Step 3: Implement routine state changes and wandering**

Enable component ticking only while `Wandering` and unpaused. Use `UNavigationSystemV1::GetRandomReachablePointInRadius`; on a valid point call `AAIController::MoveToLocation`. Track whether the component owns the request, broadcast start/completion/failure, and schedule the next selection after a short configurable idle delay. `SetRoutineState` stops routine-owned movement when leaving `Wandering` and never rebroadcasts an unchanged state.

- [ ] **Step 4: Implement directed scene move**

Bind once to `AAIController::ReceiveMoveCompleted`, store the scene's request ID, and ignore unrelated completions. Treat rejected requests and unsuccessful path results as `Failed`, broadcast failure, and call rollback. On success, verify distance to anchor is within `AcceptanceRadius`, apply anchor facing/configured routine/interaction dialogue, and mark `Active`.

- [ ] **Step 5: Build and verify routine/move tests GREEN**

Run build and both focused commands from Step 2. Expected: all state, pause, radius, arrival, and failure assertions pass.

- [ ] **Step 6: Commit Task 5 files**

```powershell
git add Source/Dispatch/Public/Component/NPCRoutineComponent.h Source/Dispatch/Private/Component/NPCRoutineComponent.cpp Source/Dispatch/Public/NPC/NPCSceneSubsystem.h Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp Source/Dispatch/Dispatch.Build.cs Source/Dispatch/Private/Tests/NPCSystemTests.cpp Source/Dispatch/Private/Tests/NPCSceneTests.cpp
git commit -m "feat: add lightweight NPC movement routines"
```

### Task 6: Entry Dialogue, Routine Restoration, and Scene Replacement

**Files:**
- Modify: `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h`
- Modify: `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCSceneTests.cpp`

**Interfaces:**
- Consumes: `StartDialogueForNPC`, `OnDialogueStartedForNPCNative`, and `OnDialogueEndedForNPCNative`.
- Produces: `WaitingForEntryDialogue -> Active`, exact NPC/dialogue matching, and stale-callback-safe replacement.

- [ ] **Step 1: Write failing orchestration tests**

Use an in-memory dialogue table. Assert:

- entry ID starts automatically while the component retains the distinct interaction ID;
- `bWaitForEntryDialogue=true` reports `WaitingForEntryDialogue` until the matching dialogue ends;
- no entry ID reports `Active` immediately;
- a failed entry start returns false and restores transform/behavior/routine/dialogue;
- ending another NPC's dialogue does not advance this scene;
- ending the matching dialogue restores configured `Wandering` or `Sitting` and reports `Active`;
- starting a second scene for the same NPC removes old callbacks, stops old movement, and exposes only the second `SceneID`;
- a late completion carrying the old ID cannot mutate the replacement scene.

- [ ] **Step 2: Run focused tests to verify RED**

Build and run `Automation RunTests Dispatch.NPC.Scene.Dialogue;Quit`. Expected: entry dialogue and replacement assertions fail.

- [ ] **Step 3: Implement dialogue phase and identity checks**

At arrival, set the configured routine/interaction ID. If entry is present, call `StartDialogueForNPC` and store both entry ID and NPC weak pointer; the dialogue subsystem snapshots the configured idle routine and temporarily applies `Talking`. On start failure, rollback and set failure. Use the NPC-aware start delegate to pause wandering for any dialogue involving an active-scene NPC. In the native end handler, require all of the following before advancing an entry-dialogue wait:

```cpp
Record.NPC.Get() == EndedNPC
Record.EntryDialogueTreeID == EndedDialogueTreeID
Record.SceneID == GetActiveSceneID(Record.NPCID)
```

For any ended dialogue whose NPC owns an active scene, unpause routine movement after `UDialogueSubsystem` has restored its snapshot. Only the exact entry ID changes `WaitingForEntryDialogue` to `Active`. Remove the record before replacing a scene so stale callbacks cannot match the replacement's scene ID.

- [ ] **Step 4: Build and verify dialogue scene tests GREEN**

Run build and `Automation RunTests Dispatch.NPC.Scene.Dialogue;Quit`. Expected: every entry/interaction, failure, restoration, and replacement assertion passes.

- [ ] **Step 5: Commit Task 6 files**

```powershell
git add Source/Dispatch/Public/NPC/NPCSceneSubsystem.h Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp Source/Dispatch/Private/Tests/NPCSceneTests.cpp
git commit -m "feat: orchestrate NPC scene dialogue"
```

### Task 7: Unified StateTree Task

**Files:**
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp`

**Interfaces:**
- Produces: `FStoryStateTreePlayNPCSceneTaskInstanceData` with editable `SceneID` and transient `ActiveNPCID`.
- Produces: `FStoryStateTreePlayNPCSceneTask`, displayed as `Story: Play NPC Scene`.

- [ ] **Step 1: Write failing StateTree API and pure status-mapping tests**

Assert the task display name, that `SceneID` is the only editable parameter, and that `bShouldCallTick` is true. Add a pure helper on the task so all paths can be tested without constructing a live StateTree execution context:

```cpp
TestEqual(TEXT("Moving waits"), Task.MapSceneStatus(ENPCSceneStatus::Moving), EStateTreeRunStatus::Running);
TestEqual(TEXT("Dialogue waits"), Task.MapSceneStatus(ENPCSceneStatus::WaitingForEntryDialogue), EStateTreeRunStatus::Running);
TestEqual(TEXT("Active succeeds"), Task.MapSceneStatus(ENPCSceneStatus::Active), EStateTreeRunStatus::Succeeded);
TestEqual(TEXT("Failure fails"), Task.MapSceneStatus(ENPCSceneStatus::Failed), EStateTreeRunStatus::Failed);
```

Add a subsystem-backed task test fixture that covers rejected start, immediate teleport success, pending move, pending dialogue, completion, replacement mismatch, and interrupted cleanup with both exit policies.

- [ ] **Step 2: Build/run to verify RED**

Build and run `Automation RunTests Dispatch.Story.Flow.NPCSceneTask;Quit`. Expected: missing task/data/helper.

- [ ] **Step 3: Implement the task**

`EnterState` validates `SceneID`, gets `UNPCSceneSubsystem` from the director world, calls `PlayNPCScene`, stores `GetNPCIDForScene(SceneID)`, then maps current status. `Tick` first verifies the stored NPC still owns the same `SceneID`, then maps status. `ExitState` checks `Transition.CurrentRunStatus != EStateTreeRunStatus::Succeeded`; if the stored NPC still owns the pending scene, call `AbortNPCScene(ActiveNPCID)`, which applies the row's `ExitPolicy`.

Update the legacy `Start Dialogue` node to call `StartDialogueForNPC` for an `ANPCCharacter`, retaining `StartDialogue` for a null/non-NPC target and leaving the legacy global wait node unchanged.

- [ ] **Step 4: Build and verify StateTree tests GREEN**

Run build and `Automation RunTests Dispatch.Story.Flow.NPCSceneTask;Quit`, then `Automation RunTests Dispatch.Story.Flow.StateTree;Quit`. Expected: new task paths and existing node API tests pass.

- [ ] **Step 5: Commit Task 7 files**

```powershell
git add Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp
git commit -m "feat: add play NPC scene StateTree task"
```

### Task 8: Full Verification and Manual Handoff

**Files:**
- Modify only if verification exposes a tested defect in files already listed above.

**Interfaces:**
- Verifies all specified runtime and compatibility behavior.

- [ ] **Step 1: Re-read the design and create a requirement checklist**

Check each design heading against a production path and an automation assertion: data, anchors, lookup, keep/teleport/move, routine events, wandering pause/resume, NPC-scoped dialogue, entry/interaction separation, replacement, two exit policies, StateTree statuses, logging, and editor settings.

- [ ] **Step 2: Build DispatchEditor from a clean command invocation**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: exit code `0` and `Result: Succeeded`.

- [ ] **Step 3: Run the complete NPC suite**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.NPC;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: all `Dispatch.NPC` tests successful, zero failed tests.

- [ ] **Step 4: Run the complete Dialogue suite**

Run the same editor command with `Automation RunTests Dispatch.Dialogue;Quit`. Expected: all dialogue tests successful, zero failed tests.

- [ ] **Step 5: Run the complete Story Flow suite**

Run the same editor command with `Automation RunTests Dispatch.Story.Flow;Quit`. Expected: all story-flow tests successful, zero failed tests.

- [ ] **Step 6: Inspect final scope**

Run `git status --short` and `git diff --stat`. Confirm no `.uasset` files, report-system runtime files, or unrelated user files are included. Review warnings/errors in the three fresh automation logs.

- [ ] **Step 7: Commit any verified final corrections**

If Step 2-6 required a test-backed correction, stage only its listed source/test files and commit with `fix: complete NPC scene verification`. If no correction was required, do not create an empty commit.

- [ ] **Step 8: Hand off the exact manual editor work**

Report that the user must create/configure `DT_NPCScenes`, `DT_NPCDialogue`, `BP_NPCSceneAnchor`, map anchors, unique NPC IDs, Blueprint/AnimBP responses for routine events, sitting/wandering presentation, the `Story: Play NPC Scene` nodes, and both Dispatch Content table settings.
