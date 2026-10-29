# Fixed Story Compass Guidance Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add opt-in fixed-location compass markers and let story StateTrees pause until a matching gameplay signal is submitted.

**Architecture:** Fixed actors register stable IDs with a world subsystem; `AStoryDirector` owns the active ID; a C++ `UCompassWidget` resolves and renders it. Generic StateTree tasks show/hide the marker and wait for filtered story signals. Mailbox interaction submits the opened letter ID through the existing unified story-signal entry point.

**Tech Stack:** Unreal Engine 5.6, C++20/Unreal reflection, UMG, StateTree, Gameplay Tags, Unreal Automation Tests.

## Global Constraints

- Preserve all current project modifications and unrelated assets.
- Do not upload to GitHub or create Git commits.
- Put reusable runtime behavior in C++; Blueprint and StateTree only configure visuals, parameters, conditions, and sequence.
- Use incremental `DispatchEditor` compilation and focused automation tests.
- Fixed locations only; moving NPC guidance, distance dismissal, and automatic timed hints are out of scope.

---

### Task 1: Fixed target registry

**Files:**
- Create: `Source/Dispatch/Public/Story/Navigation/StoryNavigationTargetComponent.h`
- Create: `Source/Dispatch/Private/Story/Navigation/StoryNavigationTargetComponent.cpp`
- Create: `Source/Dispatch/Public/Story/Navigation/StoryNavigationSubsystem.h`
- Create: `Source/Dispatch/Private/Story/Navigation/StoryNavigationSubsystem.cpp`
- Create: `Source/Dispatch/Private/Tests/StoryNavigationTests.cpp`

**Interfaces:**
- Produces: `UStoryNavigationTargetComponent::TargetID`, `UStoryNavigationSubsystem::RegisterTarget`, `UnregisterTarget`, and `ResolveTargetLocation(FName, FVector&)`.

- [ ] Write automation tests proving a non-empty ID resolves its owner's location, unregistration removes it, empty IDs are ignored, and duplicate IDs keep a deterministic registered target while warning.
- [ ] Run `Dispatch.Story.Navigation` and verify the new test fails because the types do not exist.
- [ ] Implement the component lifecycle registration and world-subsystem weak registry with no ticking.
- [ ] Run `Dispatch.Story.Navigation` and verify it passes.

### Task 2: Story-owned active marker and StateTree show/hide tasks

**Files:**
- Modify: `Source/Dispatch/Public/Story/StoryDirector.h`
- Modify: `Source/Dispatch/Private/Story/StoryDirector.cpp`
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryHUDStatusTests.cpp`

**Interfaces:**
- Produces: `AStoryDirector::SetObjectiveMarkerTarget(FName)`, `ClearObjectiveMarkerTarget()`, `GetObjectiveMarkerTarget() const`.
- Produces: `FStoryStateTreeShowObjectiveMarkerTask` and `FStoryStateTreeHideObjectiveMarkerTask`.

- [ ] Add failing tests for director marker persistence, clear behavior, task display names, and `TargetID` instance-data exposure.
- [ ] Run `Dispatch.Story.HUDStatus` and verify the new assertions fail.
- [ ] Implement the director API and the two immediate StateTree tasks.
- [ ] Run `Dispatch.Story.HUDStatus` and verify it passes.

### Task 3: Filtered wait-for-signal task

**Files:**
- Modify: `Source/Dispatch/Public/Story/StoryDirector.h`
- Modify: `Source/Dispatch/Private/Story/StoryDirector.cpp`
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Create: `Source/Dispatch/Private/Tests/StorySignalWaitTests.cpp`

**Interfaces:**
- Produces: monotonically increasing `AStoryDirector::GetSubmittedSignalSerial()` and current signal access.
- Produces: `FStoryStateTreeWaitForSignalTaskInstanceData` with required `Trigger`, optional `SourceID`, optional `ContextName`, optional enabled `ContextValue`, and runtime starting serial.
- Produces: pure `Matches(const FStorySignal&, const InstanceData&)` for focused tests.

- [ ] Add failing tests for exact trigger matching, optional source/context behavior, context-value opt-in, and serial advancement on submission.
- [ ] Run `Dispatch.Story.SignalWait` and verify failure before implementation.
- [ ] Implement the director serial and ticking task. Record the serial on entry; succeed only on a newer matching signal; remain running for unrelated signals.
- [ ] Run `Dispatch.Story.SignalWait` and verify it passes.

### Task 4: Reusable mailbox-open signal

**Files:**
- Modify: `Config/DefaultGameplayTags.ini`
- Modify: `Source/Dispatch/Private/Items/Mailbox.cpp`
- Modify: `Source/Dispatch/Private/Tests/MailboxTests.cpp`

**Interfaces:**
- Produces signal: Trigger `Story.Trigger.MailOpened`, SourceID `Mailbox`, ContextName equal to the delivered `LetterItemInfo.ItemID`.

- [ ] Add a focused test for the registered gameplay tag and for preserving the taken Letter ID across the interaction path.
- [ ] Run `Dispatch.Items.Mailbox` and verify the new expectation fails.
- [ ] Register `Story.Trigger.MailOpened`; after `OnLetterTaken` returns, submit the generic signal using the captured taken letter ID, then empty the mailbox.
- [ ] Run `Dispatch.Items.Mailbox` and verify it passes.

### Task 5: C++ compass widget runtime

**Files:**
- Create: `Source/Dispatch/Public/Widget/CompassWidget.h`
- Create: `Source/Dispatch/Private/Widget/CompassWidget.cpp`
- Create: `Source/Dispatch/Private/Tests/CompassWidgetTests.cpp`

**Interfaces:**
- Produces: `UCompassWidget` with optional UMG bindings `Text_West`, `Text_South`, `Text_North`, `Text_East`, and `Image_ObjectiveMarker`.
- Produces: pure angle-to-offset/visibility helpers used by cardinal labels and the objective marker.

- [ ] Add failing tests for wrap-around normalization, cardinal offsets, marker offset, and visibility boundaries.
- [ ] Run `Dispatch.Widget.Compass` and verify failure before the class exists.
- [ ] Implement `NativeConstruct`/`NativeTick`, player yaw lookup, cardinal label updates, director marker lookup, target registry resolution, and safe hiding for missing dependencies.
- [ ] Run `Dispatch.Widget.Compass` and verify it passes.

### Task 6: Blueprint and StateTree asset wiring

**Files:**
- Modify: `Content/Blueprints/Widget/WBP_Compass.uasset`
- Modify: the fixed mailbox Blueprint or placed mailbox actor containing the navigation target component
- Modify: `Content/Blueprints/Story/StateTrees/ST_Day01.uasset`

**Interfaces:**
- Consumes: `UCompassWidget`, `UStoryNavigationTargetComponent`, and the three new StateTree tasks.

- [ ] Reparent `WBP_Compass` to `UCompassWidget` while preserving its designer hierarchy and styling.
- [ ] Add `Image_ObjectiveMarker` under `CompassRoot`; default it to collapsed/hidden and configure its art in UMG.
- [ ] Remove the migrated Blueprint `Event Construct`, `Event Tick`, and `UpdateCompassText` runtime graph.
- [ ] Add a navigation target component to the fixed mailbox and set `TargetID = Location.Mailbox.Main`.
- [ ] Change Day 1 to `Deliver Letter -> Set Objective -> Show Objective Marker -> Wait For Signal -> Hide Objective Marker -> next story step`, filtering `MailOpened` by `ContextName = Letter_Day01_Intro`.
- [ ] Compile and save only the touched Blueprint and StateTree assets; inspect their compiler results for errors.

### Task 7: Focused verification

**Files:**
- Verify only; do not modify unrelated files.

- [ ] Run the incremental `DispatchEditor Win64 Development` build and require exit code 0.
- [ ] Run `Dispatch.Story.Navigation`, `Dispatch.Story.HUDStatus`, `Dispatch.Story.SignalWait`, `Dispatch.Items.Mailbox`, and `Dispatch.Widget.Compass` only.
- [ ] Launch the project headlessly once with `-ExecCmds=Quit -unattended -nullrhi` to catch asset-load and Blueprint-parent errors.
- [ ] Review `git diff` and `git status --short`; confirm no unrelated file was overwritten, no commit exists, and no remote action occurred.
