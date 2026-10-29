# Seven-Day Reactive Story System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an empty, Data Asset-configurable C++ framework for seven-day signals, conditions, state, event dispatch, dialogue selection, map responders, and endings.

**Architecture:** A placeable day-flow actor submits gameplay-tag signals to the existing story director. The director evaluates Data Asset rules through instanced condition objects and runs one Blueprint event at a time. A Game Instance subsystem stores only value state; optional dialogue and map response assets query that state without hard-coded story content.

**Tech Stack:** Unreal Engine 5.6, C++17, Gameplay Tags, UMG-compatible Blueprint APIs, Unreal Automation Tests.

## Global Constraints

- Do not add authored days, events, dialogue, CG, sounds, NPCs, or map assets.
- Do not modify, stage, revert, or delete existing user changes outside the listed source files.
- Do not move `StoryDirector` or `StoryEventBase`.
- Add `GameplayTags` to `Dispatch.Build.cs`.
- No remote push or GitHub upload.

---

### Task 1: Runtime State and Extensible Conditions

**Files:**
- Modify: `Source/Dispatch/Dispatch.Build.cs`
- Create: `Source/Dispatch/Public/Story/State/StoryStateTypes.h`
- Create: `Source/Dispatch/Public/Story/State/StoryWorldStateSubsystem.h`
- Create: `Source/Dispatch/Private/Story/State/StoryWorldStateSubsystem.cpp`
- Create: `Source/Dispatch/Public/Story/Rules/StoryConditionBase.h`
- Create: `Source/Dispatch/Private/Story/Rules/StoryConditionBase.cpp`
- Test: `Source/Dispatch/Private/Tests/StoryWorldStateTests.cpp`

**Interfaces:**
- Produces `UStoryWorldStateSubsystem::AddTag`, `RemoveTag`, `HasTag`, `SetValue`, `AddValue`, `GetValue`, `SetNamedResult`, `GetNamedResult`, `MarkConsumed`, and `IsConsumed`.
- Produces `FStorySignal`, `EStoryConditionMatchMode`, and `UStoryConditionBase::Matches`.

- [ ] **Step 1: Write failing automation tests** for tags, integer values, named results, consumed IDs, empty conditions, and `MatchAll`/`MatchAny`.
- [ ] **Step 2: Run `Dispatch.Story.State`** and verify the classes are unavailable.
- [ ] **Step 3: Add `GameplayTags` and implement the subsystem plus abstract condition base**. Keep all state value-only and broadcast `OnStoryStateChanged` after mutations.
- [ ] **Step 4: Add native day/tag/value/result/consumed condition subclasses** and condition-list evaluation.
- [ ] **Step 5: Re-run `Dispatch.Story.State`** and verify all tests pass.

### Task 2: Configurable Rule Assets and Day Flow

**Files:**
- Create: `Source/Dispatch/Public/Story/Flow/DayFlowTypes.h`
- Create: `Source/Dispatch/Public/Story/Flow/DayDefinition.h`
- Create: `Source/Dispatch/Private/Story/Flow/DayDefinition.cpp`
- Create: `Source/Dispatch/Public/Story/Flow/DayFlowManager.h`
- Create: `Source/Dispatch/Private/Story/Flow/DayFlowManager.cpp`
- Create: `Source/Dispatch/Public/Story/Rules/StoryEventDefinition.h`
- Create: `Source/Dispatch/Private/Story/Rules/StoryEventDefinition.cpp`
- Create: `Source/Dispatch/Public/Story/Rules/StoryDialogueSet.h`
- Create: `Source/Dispatch/Private/Story/Rules/StoryDialogueSet.cpp`
- Create: `Source/Dispatch/Public/Story/Rules/StoryEndingDefinition.h`
- Create: `Source/Dispatch/Private/Story/Rules/StoryEndingDefinition.cpp`
- Test: `Source/Dispatch/Private/Tests/DayFlowManagerTests.cpp`

**Interfaces:**
- Produces `UDayDefinition`, `UStoryEventDefinition`, `UStoryDialogueSet`, and `UStoryEndingDefinition` Data Asset classes.
- Produces `ADayFlowManager::StartDayFlow`, `RegisterCompletedWork`, `TryEndCurrentDay`, and `CompleteDayTransition`.

- [ ] **Step 1: Write failing tests** for unique work IDs, day start, rejected sleep, transition completion, and terminal day-seven delegate.
- [ ] **Step 2: Implement empty Data Asset schemas** with editable arrays, soft references, enabled flags, and no authored content.
- [ ] **Step 3: Implement the placeable flow manager**; it emits tags/signals/delegates but never plays presentation itself.
- [ ] **Step 4: Re-run `Dispatch.DayFlow`** and verify lifecycle tests pass.

### Task 3: Queue-Based Story Director

**Files:**
- Modify: `Source/Dispatch/Public/Story/StoryDirector.h`
- Modify: `Source/Dispatch/Private/Story/StoryDirector.cpp`
- Modify: `Source/Dispatch/Public/Story/StoryEventBase.h`
- Modify: `Source/Dispatch/Private/Story/StoryEventBase.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryEventSystemTests.cpp`

**Interfaces:**
- Produces `AStoryDirector::SubmitStorySignal`, `ResolveEnding`, and a queue that preserves existing `TryStartStoryEvent`.
- `AStoryEventBase::EndPlay` informs the director about abnormal destruction.

- [ ] **Step 1: Write failing tests** for day-gated signal matching, disabled rules, priority ordering, one-shot behavior, blocking queue order, and destroyed-event recovery.
- [ ] **Step 2: Add an editable event catalog and evaluate only matching day definitions** using the state subsystem and condition objects.
- [ ] **Step 3: Implement queue start, completion, and abort handling** without ticking.
- [ ] **Step 4: Re-run `Dispatch.Story.Director`** and verify legacy and new tests pass.

### Task 4: Conditional Dialogue and Generic Map Responders

**Files:**
- Modify: `Source/Dispatch/Public/Component/NPCStoryMemoryComponent.h`
- Modify: `Source/Dispatch/Private/Component/NPCStoryMemoryComponent.cpp`
- Create: `Source/Dispatch/Public/Story/World/StoryStateResponder.h`
- Create: `Source/Dispatch/Private/Story/World/StoryStateResponder.cpp`
- Create: `Source/Dispatch/Public/Story/World/StoryResponseDefinition.h`
- Create: `Source/Dispatch/Private/Story/World/StoryResponseDefinition.cpp`
- Create: `Source/Dispatch/Public/Story/World/StoryWorldFeedbackManager.h`
- Create: `Source/Dispatch/Private/Story/World/StoryWorldFeedbackManager.cpp`
- Test: `Source/Dispatch/Private/Tests/StoryDialogueAndWorldResponseTests.cpp`

**Interfaces:**
- Produces `UStoryDialogueSet::ResolveLine` and optional `UNPCStoryMemoryComponent::DialogueSet`.
- Produces placeable `AStoryStateResponder` with `ResponderID` and `OnStoryResponseStateApplied`.

- [ ] **Step 1: Write failing tests** for dialogue priority/fallback and one-shot consumption, responder duplicate-ID rejection, and response priority.
- [ ] **Step 2: Implement dialogue resolution while retaining current hard-coded fallback lines** when no asset is assigned or no entry matches.
- [ ] **Step 3: Implement generic responder discovery and response-state application**; response state names are passed to Blueprint and have no prescribed meaning.
- [ ] **Step 4: Re-run `Dispatch.Story.Dialogue` and `Dispatch.Story.World`** and verify all tests pass.

### Task 5: Validation and Verification

**Files:**
- Modify: feature-specific test files created above

- [ ] **Step 1: Add reflection tests** confirming Data Asset fields, condition arrays, event catalog, responder ID, and Blueprint APIs are editable.
- [ ] **Step 2: Build** with `Build.bat DispatchEditor Win64 Development -Project=... -WaitMutex -FromMsBuild`.
- [ ] **Step 3: Run focused automation suites** `Dispatch.Story` and `Dispatch.DayFlow` in `UnrealEditor-Cmd`.
- [ ] **Step 4: Inspect `git diff --check` and the changed-file list** to ensure only planned source files and tests changed.
