# Quest Writing Prototype Implementation Plan

> **For agentic workers:** Execute inline in the current workspace. Do not create commits or push.

**Goal:** Build the smallest playable flow where StateTree publishes one quest, the player accepts it at the notice board, writes a three-section report at a separate desk, seals it, and lets StateTree continue.

**Architecture:** A `UGameInstanceSubsystem` owns the single active quest and its runtime state. StateTree publishes inline prototype quest data and waits for a sealed-report story signal. The existing notice board reads and accepts the active quest; a new writing desk opens a C++ widget base that records one choice per section and seals the report.

**Tech Stack:** Unreal Engine 5.6, C++, UMG, StateTree, Gameplay Tags, Automation Tests.

## Global Constraints

- Keep existing local commits and history.
- Do not push or create a commit.
- Prefer reusable runtime logic in C++.
- StateTree supplies parameters and controls story order.
- Build `DispatchEditor` incrementally and run only related automation tests.
- Prototype scope excludes NPC dispatch, items, evidence acquisition, persistence, and final visual polish.

---

### Task 1: Active quest runtime state

**Files:**
- Modify: `Source/Dispatch/Public/Quest/QuestTypes.h`
- Create: `Source/Dispatch/Public/Quest/QuestSubsystem.h`
- Create: `Source/Dispatch/Private/Quest/QuestSubsystem.cpp`
- Create: `Source/Dispatch/Private/Tests/QuestSubsystemTests.cpp`

**Interfaces:**
- `FQuestWritingSection`: section ID, heading, selectable option texts.
- `FQuestDefinition`: existing quest display fields plus writing sections.
- `UQuestSubsystem::PublishQuest`, `AcceptActiveQuest`, `SelectWritingOption`, `SealActiveReport`.
- Query functions expose active definition, state, selections, and whether writing is available.

- [ ] Write automation tests for publishing, accepting, rejecting invalid selections, selecting one option per section, and sealing only when complete.
- [ ] Run `Dispatch.Quest.Prototype` and verify the new tests fail before implementation.
- [ ] Implement the minimal subsystem state machine: None, Available, Accepted, Sealed.
- [ ] Run `Dispatch.Quest.Prototype` and verify all subsystem tests pass.

### Task 2: StateTree publication and completion

**Files:**
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp`

**Interfaces:**
- `Story: Publish Quest` takes an inline `FQuestDefinition` and calls `UQuestSubsystem::PublishQuest`.
- Sealing submits a story signal whose trigger is configurable on the quest definition and whose `SourceID` is the quest ID.
- Existing `Story: Wait For Signal` waits for that signal; no second custom wait node is required.

- [ ] Add focused task tests for publication validation and quest state changes.
- [ ] Run the focused StateTree test and confirm failure.
- [ ] Implement `FStoryStateTreePublishQuestTask`.
- [ ] Run the focused StateTree and quest tests.

### Task 3: Notice board bridge

**Files:**
- Modify: `Source/Dispatch/Public/Items/NoticeBoard.h`
- Modify: `Source/Dispatch/Private/Items/NoticeBoard.cpp`
- Modify: `Source/Dispatch/Private/Tests/NoticeBoardTests.cpp`

**Interfaces:**
- `GetActiveQuestInfo()` exposes the subsystem-owned quest to Blueprint UI.
- `AcceptActiveQuest()` accepts the subsystem-owned quest.
- Existing manually filled `QuestList` remains temporarily for asset compatibility but is no longer authoritative when a subsystem quest is active.

- [ ] Add tests proving the board reflects and accepts a published subsystem quest.
- [ ] Run the notice-board test and verify failure.
- [ ] Add the minimal bridge methods and refresh Niagara from subsystem state.
- [ ] Run `Dispatch.Quest.Prototype` and `Dispatch.Items.NoticeBoard` tests.

### Task 4: Writing widget and independent desk

**Files:**
- Create: `Source/Dispatch/Public/Widget/QuestWritingWidget.h`
- Create: `Source/Dispatch/Private/Widget/QuestWritingWidget.cpp`
- Create: `Source/Dispatch/Public/Items/QuestWritingDesk.h`
- Create: `Source/Dispatch/Private/Items/QuestWritingDesk.cpp`
- Create: `Source/Dispatch/Private/Tests/QuestWritingTests.cpp`

**Interfaces:**
- `UQuestWritingWidget` exposes Blueprint-callable getters, selection, seal, and close actions; Blueprint implements layout and forwards button clicks.
- `AQuestWritingDesk` implements `IInteractInterface`, owns mesh/box/camera components, opens the configured widget only when the active quest is accepted, and restores player input when closed.
- If no Blueprint widget class is configured, the actor fails safely without changing player input.

- [ ] Add tests for writing availability, section selection replacement, completion state, and seal rejection when incomplete.
- [ ] Run focused tests and confirm failure.
- [ ] Implement the widget logic and desk interaction shell.
- [ ] Run focused tests.

### Task 5: Integration verification and Blueprint handoff

**Files:**
- Modify only if required by compilation: `Source/Dispatch/Dispatch.Build.cs`

- [ ] Incrementally build `DispatchEditor Win64 Development`.
- [ ] Run only `Dispatch.Quest.Prototype`, related notice-board tests, and related StateTree tests.
- [ ] Report the exact Blueprint steps: create `BP_QuestWritingDesk`, create `WBP_QuestWriting`, reconnect `BP_QuestNotice` to subsystem-backed getters/acceptance, and add `Story: Publish Quest` plus `Story: Wait For Signal` to `ST_Day01`.
- [ ] Confirm no commit or push was created.
