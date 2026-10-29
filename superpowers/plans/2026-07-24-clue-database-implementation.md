# Data-Driven Readable Clues Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace StateTree-authored notice text and clue markup with ID-driven quest/document/clue data, while making clue collection reusable across all readable text.

**Architecture:** Three configured DataTables provide quest, document, and clue definitions. `UClueSubsystem` owns collected IDs; `UQuestContentSubsystem` resolves quest/document rows and generates internal rich-text markup. `UClueRichTextBlock` consumes the internal format, hides undiscovered clue affordances, and renders collected spans grey and disabled.

**Tech Stack:** Unreal Engine 5.6 C++, UGameInstanceSubsystem, UDeveloperSettings, UDataTable, UMG/Slate rich text, StateTree, Unreal Automation Tests.

## Global Constraints

- StateTree authors only `SlotIndex`, `QuestID`, and `State`.
- Authors never type `<clue>` markup.
- Undiscovered clue spans must match ordinary text in color, underline, hover, and cursor.
- Collected clue spans are grey and cannot collect again.
- Collection is independent of quest acceptance.
- Existing Blueprint assets and unrelated dirty-worktree changes must be preserved.
- Report arrangement, dispatch outcome evaluation, clue journal UI, and SaveGame are outside this increment.

---

### Task 1: Content Rows, Settings, and Document Builder

**Files:**
- Create: `Source/Dispatch/Public/Quest/ReadableContentTypes.h`
- Create: `Source/Dispatch/Public/Quest/DispatchContentSettings.h`
- Create: `Source/Dispatch/Private/Quest/DispatchContentSettings.cpp`
- Create: `Source/Dispatch/Public/Quest/QuestContentSubsystem.h`
- Create: `Source/Dispatch/Private/Quest/QuestContentSubsystem.cpp`
- Create: `Source/Dispatch/Private/Tests/QuestContentSubsystemTests.cpp`
- Modify: `Source/Dispatch/Dispatch.Build.cs`

**Interfaces:**
- Produces `FClueDefinitionRow`, `FReadableTextSegment`, `FReadableDocumentRow`, `FQuestContentRow`, and `FResolvedQuestContent`.
- Produces `UDispatchContentSettings` with soft references `QuestTable`, `DocumentTable`, and `ClueTable`.
- Produces `UQuestContentSubsystem::ResolveQuestContent(FName, FResolvedQuestContent&)`.
- Produces `UQuestContentSubsystem::BuildDocumentRichText(FName, FText&)`.

- [ ] Write tests that create transient DataTables, inject them through `SetTablesForTesting`, and assert exact row lookup, ordered rendering, escaping of `&`, `<`, `>`, and `"`, internal clue-tag generation, and invalid clue fallback to escaped plain text.
- [ ] Build and verify failure because the row types and subsystem do not exist.
- [ ] Implement the five Blueprint-visible structs. DataTable row names remain the sole IDs; no row contains a duplicate ID field.
- [ ] Implement `UDispatchContentSettings : UDeveloperSettings` and add `DeveloperSettings` to `Dispatch.Build.cs`.
- [ ] Implement `EscapeRichText`, document lookup, clue-row validation, and quest resolution. Generated clue markup has exactly `<clue id="ID">escaped text</>`.
- [ ] Build and run `Dispatch.Quest.Content`; expect all tests to pass.

---

### Task 2: Independent Clue Runtime State

**Files:**
- Create: `Source/Dispatch/Public/Quest/ClueSubsystem.h`
- Create: `Source/Dispatch/Private/Quest/ClueSubsystem.cpp`
- Create: `Source/Dispatch/Private/Tests/ClueSubsystemTests.cpp`
- Modify: `Source/Dispatch/Public/Quest/QuestSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/QuestSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/QuestPrototypeTests.cpp`

**Interfaces:**
- Produces `CollectClue`, `HasClue`, `HasClueDefinition`, `FindClueDefinition`, and `GetCollectedClueIDs` on `UClueSubsystem`.
- Produces Blueprint event `OnClueCollected` and native test/C++ event `OnClueCollectedNative`.
- `UQuestSubsystem` delegates its compatibility methods and writing unlock checks to `UClueSubsystem`.

- [ ] Write tests for missing table, empty ID, unknown ID, first collection, duplicate collection, broadcast count, definition lookup, and collection before quest publication or acceptance.
- [ ] Build and verify the tests fail on missing `UClueSubsystem`.
- [ ] Implement `UClueSubsystem : UGameInstanceSubsystem` using `UDispatchContentSettings::ClueTable`, with a transient table override under `WITH_DEV_AUTOMATION_TESTS`.
- [ ] Remove authoritative `CollectedClueIDs` ownership from `UQuestSubsystem`; delegate existing Blueprint functions and `IsOptionUnlocked` to the clue subsystem. Add a test-only subsystem override so existing direct-object tests remain deterministic.
- [ ] Update `QuestPrototypeTests` to create a clue table and inject `UClueSubsystem`.
- [ ] Build and run `Dispatch.Quest.Clue` and `Dispatch.Quest.Prototype`; expect all tests to pass.

---

### Task 3: Hidden and Collected Rich-Text Runs

**Files:**
- Modify: `Source/Dispatch/Public/Widget/ClueRichTextBlock.h`
- Modify: `Source/Dispatch/Private/Widget/ClueRichTextBlock.cpp`
- Modify: `Source/Dispatch/Private/Tests/ClueRichTextBlockTests.cpp`

**Interfaces:**
- `UClueRichTextBlock::CollectClue` calls `UClueSubsystem`.
- Produces `IsClueCollected`, `HasClueDefinition`, `BuildUndiscoveredClueStyle`, and `BuildCollectedClueStyle`.
- Undiscovered valid clues use a clickable Slate run whose cursor remains `Default`.
- Collected or invalid clues use non-clickable `FSlateTextRun`.

- [ ] Extend tests to assert the widget uses `UClueSubsystem`, exposes collected color, gives undiscovered spans the current default text style, and gives collected spans the configured grey style.
- [ ] Build and verify the new API assertions fail.
- [ ] Replace the red default clue style with the widget’s normal text style. Create a private `SHiddenClueHyperlink`/`FHiddenClueHyperlinkRun` that preserves click behavior but returns `EMouseCursor::Default`.
- [ ] During decorator creation, resolve the clue ID: valid/uncollected creates the hidden clickable run; collected creates grey `FSlateTextRun`; invalid creates ordinary `FSlateTextRun`.
- [ ] After first collection, rebuild the rich-text layout and broadcast `OnClueClicked`. Duplicate direct calls remain false at the widget layer because the span is disabled after rebuilding.
- [ ] Build and run `Dispatch.Widget.ClueRichTextBlock`; expect all tests to pass.

---

### Task 4: Quest-ID Notice Publication

**Files:**
- Modify: `Source/Dispatch/Public/Items/NoticeBoard.h`
- Modify: `Source/Dispatch/Private/Items/NoticeBoard.cpp`
- Modify: `Source/Dispatch/Public/Story/Flow/StoryStateTreeNodes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Modify: `Source/Dispatch/Private/Tests/NoticeBoardTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/StoryFlowStateTreeTests.cpp`

**Interfaces:**
- Produces `ANoticeBoard::SetQuestSlotByID(int32 SlotIndex, FName QuestID, EQuestState State)`.
- `FStoryStateTreeSetNoticeQuestTaskInstanceData` contains exactly `SlotIndex`, `QuestID`, and `State` for quest publication.
- Existing `FNoticeQuest` remains a runtime/UI view model.

- [ ] Update reflection tests to require `QuestID` and `State` and reject the old authored `Quest` property.
- [ ] Add board tests using a transient `UQuestContentSubsystem` table fixture and a test-only resolver override on `ANoticeBoard`.
- [ ] Build and verify failures before changing production code.
- [ ] Implement `SetQuestSlotByID`: resolve `FResolvedQuestContent`, construct `FNoticeQuest`, and reuse `SetQuestSlotData`.
- [ ] Replace the StateTree instance `FNoticeQuest Quest` property with `FName QuestID` and `EQuestState State`; update `EnterState` to call `SetQuestSlotByID`.
- [ ] Build and run `Dispatch.Items.NoticeBoard` and `Dispatch.Story.Flow.StateTree`; expect all tests to pass.

---

### Task 5: Regression and Editor Asset Migration

**Files:**
- Create in Unreal Editor: `DT_Quests`, `DT_Documents`, `DT_Clues`
- Modify in Unreal Editor: `ST_Day01`
- Modify: `Config/DefaultGame.ini`

**Interfaces:**
- Configures the three soft table references in Project Settings → Game → Dispatch Content.
- Migrates `PublishQuest01` to `QuestID = Quest.Day01.MerchantMissing`, `SlotIndex = 0`, `State = Available`.

- [ ] Build `DispatchEditor` and run `Dispatch.Quest`, `Dispatch.Widget.ClueRichTextBlock`, `Dispatch.Items.NoticeBoard`, and `Dispatch.Story.Flow.StateTree`.
- [ ] In the editor create the three DataTables with the new row structs and configure them in project settings.
- [ ] Add clue row `Clue.Day01.MerchantMissing`, a document row with ordered plain/clue/plain segments, and quest row `Quest.Day01.MerchantMissing` referencing that document.
- [ ] Compile `ST_Day01`, replace the removed inline quest struct with the new `QuestID`, and save.
- [ ] In PIE verify unchanged notice wording, hidden clue appearance, first-click collection, grey disabled redisplay, and collection before acceptance.
- [ ] Run `git diff --check` and inspect `git status --short`; do not stage unrelated Blueprint changes.

