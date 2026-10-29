# Notebook tab interaction

## Outcome

Implemented the split gold quest tab interaction in `UPlayerNotebookWidget`. The fixed root stays still; only `Image_Body` moves. Hover and selection use X=-4, press uses X=-1, and idle uses X=0.

The native setup removes the default button brush, keeps both image layers out of hit testing, and adds handlers without removing the Blueprint click chain that switches pages and refreshes quest content.

## Verification

- `DispatchEditor Win64 Development`: passed.
- `Dispatch.Widget.Notebook.TabInteraction`: 2 tests passed.
- The asset-loading test created the real `WBP_PlayerNotebook` and verified widget names, transparent style, layer visibility, and selected offset.

## Handoff

No `.uasset` was saved by this work. Restart the already-open editor so it loads the newly built module, then perform final visual mouse testing at the target DPI. The current implementation intentionally covers the gold quest prototype; bronze and silver can reuse the same pattern after their layers are assembled.

Follow-up fix: selected and hovered previously shared X=-4, so hovering the initially selected quest tab had no visual delta. The selected-hover state now uses X=-7; a failing regression assertion reproduced the old -4 result before the change, and the full two-test filter passed afterward.

Latest user direction supersedes the left-slide treatment for the quest tab only. Its body now scales from a right-edge pivot: idle 1.00/X 0, selected 1.02/X -1, hover 1.05/X -2, pressed 0.98/X +1. The fixed root remains unchanged. Bag and clue animation work is explicitly deferred.
