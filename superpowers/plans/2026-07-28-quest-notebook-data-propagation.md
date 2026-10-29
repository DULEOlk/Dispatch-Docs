# Quest Notebook Data Propagation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Carry quest notebook location and image fields from `DT_Quests` through quest resolution and notice-board acceptance into `FQuestInfo`, while StateTree continues to author only slot, ID, and runtime state.

**Architecture:** Static presentation data lives in `FQuestContentRow`, is copied into `FResolvedQuestContent` and `FNoticeQuest`, then converted through one Blueprint-pure helper to `FQuestInfo`. Runtime state is mapped from `EQuestState`; it is not authored in the content table.

**Tech Stack:** Unreal Engine 5.6 C++, UDataTable, StateTree, Blueprint-visible structs, Unreal Automation Tests.

## Global Constraints

- Preserve the existing `FNoticeQuest` Blueprint pins and append fields rather than nesting or replacing the struct.
- Keep `FStoryStateTreeSetNoticeQuestTaskInstanceData` limited to `SlotIndex`, `QuestID`, and `State`.
- Do not modify Blueprint assets.
- Implement behavior test-first.

---

### Task 1: Static quest presentation data

**Files:**
- Modify: `Source/Dispatch/Public/Quest/ReadableContentTypes.h`
- Modify: `Source/Dispatch/Private/Quest/QuestContentSubsystem.cpp`
- Test: `Source/Dispatch/Private/Tests/QuestContentSubsystemTests.cpp`

**Interfaces:**
- Produces `QuestLocation`, `QuestListIcon`, `QuestDetailImage`, and `QuestMapImage` on `FQuestContentRow` and `FResolvedQuestContent`.

- [ ] Add resolution assertions for all four fields and run `Dispatch.Quest.Content.ResolveQuest` to observe failure.
- [ ] Add the fields to both structs and copy them in `ResolveQuestContent`.
- [ ] Rebuild and verify the focused test passes.

### Task 2: Notice-board and notebook propagation

**Files:**
- Modify: `Source/Dispatch/Public/Items/NoticeBoard.h`
- Modify: `Source/Dispatch/Private/Items/NoticeBoard.cpp`
- Test: `Source/Dispatch/Private/Tests/NoticeBoardTests.cpp`

**Interfaces:**
- Extends `FNoticeQuest` with the four static presentation fields.
- Produces `ANoticeBoard::MakeQuestInfoFromNoticeQuest(const FNoticeQuest&)` for Blueprint acceptance storage.

- [ ] Add a transient content-subsystem fixture and assertions that `SetQuestSlotByID` stores all static fields.
- [ ] Add conversion assertions for field copying and state mapping.
- [ ] Run `Dispatch.Items.NoticeBoard.QuestSlot` to observe failure.
- [ ] Implement field propagation, conversion, and state mapping.
- [ ] Build and run `Dispatch.Items.NoticeBoard`, `Dispatch.Quest`, and `Dispatch.Widget.QuestNotebook`.
