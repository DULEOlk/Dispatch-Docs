# Clickable Clue Text Design

## Scope

The first increment adds inline clue links to quest description text. A clue is visually distinct, can be clicked, and routes its `ClueID` to `UQuestSubsystem::CollectClue`. Collected-state styling, clue journals, evolving descriptions, and Blueprint UI redesigns are outside this increment.

## Existing Data Flow

`FNoticeQuest` remains the shared payload between the StateTree, notice-board UI, accepted-quest storage, and player notebook:

- `QuestID`
- `Title`
- `Description`
- `QuestMeta`
- `State`

`Description` remains `FText`. Inline clue markup travels with that text, so accepting a quest does not require a new conversion or storage path.

Example:

```text
请调查<clue id="Clue.Day01.MerchantMissing">失踪商人的最后行踪</>，并向旅店老板报告。
```

## Runtime Architecture

`UClueRichTextBlock` derives from `URichTextBlock` and is available in the UMG palette. It injects a native Slate text decorator that recognizes `clue` runs, preserves normal wrapping, and renders the run as a hyperlink using the surrounding text font plus an editable clue color and underline style.

When a run is clicked, the decorator forwards its metadata to `UClueRichTextBlock`. The widget validates the `id`, obtains `UQuestSubsystem` from the owning game instance, calls `CollectClue`, and broadcasts a Blueprint event containing the ID and collection result.

`UQuestSubsystem` becomes a `UGameInstanceSubsystem`. Its existing public quest and writing APIs remain unchanged. Valid clue collection stays idempotent: collecting an existing ID is still a successful request, while only the first collection adds state.

## Blueprint Integration

No full-screen widget is reparented. `WBP_NoticeBoardUI` and `WBP_PlayerNotebook` keep their Blueprint business logic.

In each widget, the designer replaces `Text_Description` with `UClueRichTextBlock`, retains the existing slot and layout, reconnects the existing `Set Text` node, and sets the desired font, wrapping, line height, clue color, and underline style. The widget and its ancestors must remain hit-testable.

## Failure Behavior

- Empty or absent `id` metadata does not collect a clue.
- A click without an owning game instance or quest subsystem returns failure and still broadcasts the attempted ID/result for Blueprint feedback.
- Ordinary text and descriptions without clue tags render normally.
- Existing `FNoticeQuest` Blueprint pins remain compatible.

## Verification

Automation tests cover:

- `UQuestSubsystem` is a game-instance subsystem.
- Empty clue IDs are rejected.
- Valid and duplicate clue collection are idempotent.
- Clue metadata accepts a non-empty ID and rejects missing or empty IDs.
- `UClueRichTextBlock` exposes the expected Blueprint event and remains a `URichTextBlock`.

The project is then built as `DispatchEditor`, followed by the focused quest/clue automation tests.
