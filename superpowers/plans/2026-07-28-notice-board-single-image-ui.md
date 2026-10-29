# Notice Board Single-Image UI Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give the quest notice actor and notice detail widget C++ parents, show one `QuestDetailImage` in the notice and notebook detail UIs, and remove the obsolete `QuestMapImage` without redesigning the existing quest data flow.

**Architecture:** The existing `DT_Quests -> UQuestContentSubsystem -> FNoticeQuest -> FQuestInfo` propagation remains authoritative. `AQuestNoticeActor` adapts an existing `FNoticeQuest` into the already-supported `FQuestInfo`; `UNoticeBoardQuestWidget` renders that structure into optional UMG bindings. Existing Blueprint acceptance and controller storage remain unchanged.

**Tech Stack:** Unreal Engine 5.6 C++, UMG, Blueprint child widgets, Unreal Automation Tests.

## Global Constraints

- Do not redesign StateTree, quest resolution, notice-board acceptance, or the accepted-quest array.
- Keep `QuestListIcon` for the notebook list row.
- Keep exactly one detail image field, `QuestDetailImage`.
- Preserve the user's modified Blueprint assets; C++ provides parent APIs and the final handoff lists the minimal reparent/rewire steps.
- Implement each behavior test-first.

---

### Task 1: Single-image quest presentation contract

**Files:**
- Modify: `Source/Dispatch/Public/Quest/ReadableContentTypes.h`
- Modify: `Source/Dispatch/Public/Items/NoticeBoard.h`
- Modify: `Source/Dispatch/Public/Quest/QuestTypes.h`
- Modify: `Source/Dispatch/Private/Quest/QuestContentSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Items/NoticeBoard.cpp`
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Modify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`
- Test: `Source/Dispatch/Private/Tests/QuestContentSubsystemTests.cpp`
- Test: `Source/Dispatch/Private/Tests/NoticeBoardTests.cpp`
- Test: `Source/Dispatch/Private/Tests/QuestNotebookWidgetTests.cpp`

**Interfaces:**
- Consumes: existing `QuestDetailImage` fields and propagation.
- Produces: a single detail-image contract with no `QuestMapImage` property or notebook binding.

- [ ] **Step 1: Change tests to require the single-image contract**

Remove fixtures and assertions for `QuestMapImage`; add an explicit reflected-property assertion that the obsolete field is absent and retain behavior assertions for `QuestDetailImage`.

- [ ] **Step 2: Build to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: the updated test contract fails while production code still exposes or references `QuestMapImage`.

- [ ] **Step 3: Remove the obsolete production field and binding**

Delete `QuestMapImage` from the three quest structs, propagation copies, `UPlayerNotebookWidget`, and its detail refresh/clear paths. Do not alter `QuestListIcon` or `QuestDetailImage`.

- [ ] **Step 4: Rebuild and run focused tests**

Run `Dispatch.Quest.Content.ResolveQuest`, `Dispatch.Items.NoticeBoard.QuestIDPresentationData`, and `Dispatch.Widget.QuestNotebook`.

Expected: all focused tests pass.

### Task 2: Quest notice actor C++ parent

**Files:**
- Create: `Source/Dispatch/Public/Items/QuestNoticeActor.h`
- Create: `Source/Dispatch/Private/Items/QuestNoticeActor.cpp`
- Test: `Source/Dispatch/Private/Tests/QuestNoticeUITests.cpp`

**Interfaces:**
- Consumes: `FNoticeQuest` and `ANoticeBoard::MakeQuestInfoFromNoticeQuest`.
- Produces:
  - `void SetNoticeQuestData(const FNoticeQuest& InQuest, ANoticeBoard* InSourceBoard)`
  - `FQuestInfo GetQuestInfo() const`
  - `ANoticeBoard* GetSourceNoticeBoard() const`
  - Blueprint event `OnNoticeQuestDataChanged`

- [ ] **Step 1: Write a failing behavior test**

Spawn `AQuestNoticeActor`, call `SetNoticeQuestData`, and assert the actor stores the source board and exposes the converted title, ID, state, and detail-image soft path.

- [ ] **Step 2: Run the focused test to verify RED**

Expected: build failure because `AQuestNoticeActor` does not exist.

- [ ] **Step 3: Implement the minimal actor parent**

Store the original notice data, convert with the existing helper, retain the source board as a weakly owned UObject pointer, and invoke `OnNoticeQuestDataChanged`.

- [ ] **Step 4: Build and run the actor test**

Expected: the actor adapter test passes.

### Task 3: Notice detail widget C++ parent

**Files:**
- Create: `Source/Dispatch/Public/Widget/NoticeBoardQuestWidget.h`
- Create: `Source/Dispatch/Private/Widget/NoticeBoardQuestWidget.cpp`
- Test: `Source/Dispatch/Private/Tests/QuestNoticeUITests.cpp`

**Interfaces:**
- Consumes: `FQuestInfo`.
- Produces:
  - `void DisplayQuestInfo(const FQuestInfo& InQuestInfo)`
  - `FQuestInfo GetDisplayedQuestInfo() const`
  - optional bindings `Text_Title`, `Text_Description`, `Text_Meta`, and `QuestDetailImage`

- [ ] **Step 1: Write failing widget behavior tests**

Bind real `UTextBlock`, `URichTextBlock`, and `UImage` controls to a widget instance. Assert `DisplayQuestInfo` updates all text and collapses the image when the soft texture is empty.

- [ ] **Step 2: Run the focused test to verify RED**

Expected: build failure because `UNoticeBoardQuestWidget` does not exist.

- [ ] **Step 3: Implement the minimal widget**

Store the displayed structure, set text bindings, synchronously load the existing soft texture when present, and use `Collapsed` only when no valid detail texture exists.

- [ ] **Step 4: Build and run the widget test**

Expected: the widget presentation test passes.

### Task 4: Selection layout regression and final verification

**Files:**
- Modify: `Source/Dispatch/Private/Widget/QuestListItemWidget.cpp`
- Test: `Source/Dispatch/Private/Tests/QuestNotebookWidgetTests.cpp`

**Interfaces:**
- Consumes: existing `SetQuestSelected(bool)`.
- Produces: selection icon visibility that preserves row layout when unselected.

- [ ] **Step 1: Add a failing visibility assertion**

Bind `Image_Selected`, call `SetQuestSelected(false)`, and assert the indicator is `Hidden`; call it with `true` and assert it is `HitTestInvisible`.

- [ ] **Step 2: Run the focused test to verify RED**

Expected: failure because the current unselected state is `Collapsed`.

- [ ] **Step 3: Change only the unselected visibility**

Use `ESlateVisibility::Hidden` so the selection indicator keeps its layout allocation.

- [ ] **Step 4: Build and run all affected automation tests**

Run:

```text
Dispatch.Quest.Content
Dispatch.Items.NoticeBoard
Dispatch.Widget.QuestNotebook
Dispatch.Widget.QuestNotice
Dispatch.Story.Flow.StateTree.SetNoticeQuest
```

Expected: all tests pass with zero failures.

