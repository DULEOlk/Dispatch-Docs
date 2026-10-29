# Story Objective Run Task Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a reusable Objective DataAsset and one StateTree task that owns objective display, compass guidance, signal waiting, and cleanup.

**Architecture:** `UStoryObjectiveDefinition` owns reusable content and completion filters. `AStoryDirector` owns the active objective ID and presentation. `FStoryStateTreeRunObjectiveTask` records the signal serial on entry, waits through the existing signal history, and performs ownership-safe cleanup on exit.

**Tech Stack:** Unreal Engine 5.6, C++20/Unreal reflection, StateTree, Gameplay Tags, Unreal Automation Tests.

## Global Constraints

- Preserve existing granular objective nodes and all binary assets.
- Do not edit StateTree `.uasset` files in this phase.
- Do not migrate letters, documents, NPC scenes, or dialogue in this phase.
- Use TDD and focused automation tests.
- Do not create a git commit or perform any remote action.

---

### Task 1: Objective definition and matching

**Files:**
- Create: `Source/Dispatch/Public/Story/Flow/StoryObjectiveDefinition.h`
- Create: `Source/Dispatch/Private/Story/Flow/StoryObjectiveDefinition.cpp`
- Create: `Source/Dispatch/Private/Tests/StoryObjectiveTests.cpp`

**Interfaces:**
- Produces: `UStoryObjectiveDefinition::IsDefinitionValid() const`.
- Produces: `UStoryObjectiveDefinition::MatchesSignal(const FStorySignal&) const`.

- [ ] Add a failing automation test that expects the reflected asset fields, rejects missing IDs/text/triggers and marker-without-target configuration, and verifies optional signal filters.
- [ ] Build `DispatchEditor` and verify failure because `StoryObjectiveDefinition.h` and its types do not exist.
- [ ] Implement the primary data asset, validation, and signal matching.
- [ ] Build `DispatchEditor` and run `Dispatch.Story.Objective.Definition` until it passes.

### Task 2: Director objective ownership

**Files:**
- Modify: `Source/Dispatch/Public/Story/StoryDirector.h`
- Modify: `Source/Dispatch/Private/Story/StoryDirector.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryObjectiveTests.cpp`

**Interfaces:**
- Produces: `bool AStoryDirector::BeginObjective(const UStoryObjectiveDefinition*)`.
- Produces: `void AStoryDirector::EndObjective(FName)`.
- Produces: `FName AStoryDirector::GetCurrentObjectiveID() const`.

- [ ] Extend the test first to require valid begin behavior, marker assignment, wrong-owner cleanup protection, correct-owner cleanup, and marker clearing for marker-disabled objectives.
- [ ] Build and verify the test fails because the director API is absent.
- [ ] Implement the minimal ownership API by composing the existing objective text and marker methods.
- [ ] Build and run `Dispatch.Story.Objective.Director` until it passes.

### Task 3: Run Objective StateTree task

**Files:**
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryObjectiveTests.cpp`

**Interfaces:**
- Produces: `FStoryStateTreeRunObjectiveTaskInstanceData::Objective`.
- Produces: `FStoryStateTreeRunObjectiveTask` with `EnterState`, `Tick`, and `ExitState`.

- [ ] Extend the test first to require display name `Story: Run Objective`, direct Objective asset exposure, ticking, and instance data.
- [ ] Build and verify the test fails because the task types are absent.
- [ ] Implement task entry, signal-history matching, and ownership-safe exit cleanup.
- [ ] Build and run `Dispatch.Story.Objective` until all new tests pass.

### Task 4: Regression verification

**Files:**
- Verify all modified files; do not change assets.

- [ ] Run `DispatchEditor Win64 Development` and require exit code 0.
- [ ] Run `Dispatch.Story.Objective`, `Dispatch.Story.HUDStatus`, and `Dispatch.Story.SignalWait` and require zero failed tests.
- [ ] Launch the project headlessly with `-ExecCmds=Quit -unattended -nullrhi` and inspect the log for load errors.
- [ ] Review `git diff --check`, `git diff`, and `git status --short` for unrelated changes.
