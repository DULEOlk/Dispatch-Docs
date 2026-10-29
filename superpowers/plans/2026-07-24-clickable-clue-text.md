# Clickable Clue Text Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a reusable UMG rich-text control whose inline clue links collect persistent quest clues.

**Architecture:** Keep `FNoticeQuest::Description` as tagged `FText`. A `URichTextBlock` subclass injects a native `ITextDecorator` that creates `FSlateHyperlinkRun` instances and routes their IDs to a game-instance-owned quest subsystem.

**Tech Stack:** Unreal Engine 5.6, C++20, UMG, Slate, Unreal Automation Tests.

## Global Constraints

- Do not modify `FNoticeQuest` or the accepted-quest Blueprint data flow.
- Do not edit the currently dirty Blueprint assets while implementing the C++ increment.
- Preserve every existing `UQuestSubsystem` public function signature.
- Implement behavior test-first.

---

### Task 1: Runtime clue ownership

**Files:**
- Modify: `Source/Dispatch/Public/Quest/QuestSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/QuestSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/QuestPrototypeTests.cpp`

**Interfaces:**
- Produces: `UQuestSubsystem : UGameInstanceSubsystem`
- Preserves: `bool CollectClue(FName ClueID)` and `bool HasClue(FName ClueID) const`

- [ ] Add an automation assertion that `UQuestSubsystem::StaticClass()` derives from `UGameInstanceSubsystem`.
- [ ] Add an automation assertion that collecting the same valid clue twice succeeds without creating multiple set entries.
- [ ] Run `Dispatch.Quest.Prototype` and confirm the new ownership assertion fails before the header change.
- [ ] Change the superclass include and declaration, preserving all public APIs and state.
- [ ] Make duplicate collection explicitly idempotent.
- [ ] Re-run `Dispatch.Quest.Prototype` and confirm it passes.

### Task 2: Clue metadata and rich-text control

**Files:**
- Create: `Source/Dispatch/Public/Widget/ClueRichTextBlock.h`
- Create: `Source/Dispatch/Private/Widget/ClueRichTextBlock.cpp`
- Create: `Source/Dispatch/Private/Tests/ClueRichTextBlockTests.cpp`

**Interfaces:**
- Produces: `Dispatch::ClueText::TryGetClueID(const TMap<FString, FString>&, FName&)`
- Produces: `UClueRichTextBlock : URichTextBlock`
- Produces: Blueprint event `OnClueClicked(FName ClueID, bool bCollected)`

- [ ] Add failing metadata tests for valid, missing, and empty `id`.
- [ ] Add failing reflection tests for the widget superclass and Blueprint event.
- [ ] Build `DispatchEditor` and confirm failure because the new interfaces do not exist.
- [ ] Declare the widget, metadata helper, editable clue color/underline style, event, and click-routing method.
- [ ] Implement metadata validation and game-instance subsystem routing.
- [ ] Implement an `ITextDecorator` that recognizes `<clue id="...">`, creates `FSlateHyperlinkRun`, and calls the widget.
- [ ] Override `CreateDecorators` so every instance installs the clue decorator automatically.
- [ ] Build `DispatchEditor` and run `Dispatch.Widget.ClueRichTextBlock`.

### Task 3: Regression verification and Blueprint handoff

**Files:**
- Verify only; no Blueprint asset edits.

**Interfaces:**
- Consumes: `UClueRichTextBlock` and `UQuestSubsystem`

- [ ] Build `DispatchEditor Win64 Development`.
- [ ] Run `Dispatch.Quest.Prototype`.
- [ ] Run `Dispatch.Widget.ClueRichTextBlock`.
- [ ] Run the existing notice-board automation group.
- [ ] Document the two Blueprint control replacements and the StateTree markup example in the final handoff.
