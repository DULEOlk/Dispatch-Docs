# Inventory Letter Read Button Design

## Goal

Make the action button on the item-detail page open collected letters while
remaining hidden for ordinary inventory items.

## Selected Design

`UPlayerNotebookWidget` owns the action button because it already owns inventory
selection and the right-page preview. It binds a preferred widget named
`Button_ReadLetter`, but also discovers the current designer name `Button_139`
so the user's unsaved layout works without an immediate rename. The button's
direct text child is set to `阅读`; a preferred `Text_ReadLetter` binding is also
supported.

The button is collapsed whenever selection is empty or the selected
`FDispatchItemInfo` has no valid `LetterDefinition`. Selecting a collected letter
shows the button. Clicking it creates the configured `LetterWidgetClass`, calls
`InitializeLetter` without a mailbox source, assigns the notebook as the return
widget, and adds the reader to the viewport. No inventory mutation occurs.

`ULetterWidgetBase` gains an optional return widget. Mailbox-opened letters still
restore game input when closed. Inventory-opened letters restore UI-only input,
mouse cursor, and focus to the still-visible notebook.

## Configuration

`WBP_PlayerNotebook` receives an editable `LetterWidgetClass` property which the
user sets to `WBP_LetterUI`. No Blueprint click graph is required.

## Compatibility and Safety

All UMG bindings are optional. Existing item preview, inventory selection,
mailbox collection, and legacy letter delivery remain unchanged. Codex does not
edit any `.uasset`, including the user's currently modified notebook and imported
art assets.

## Verification

Automation tests cover ordinary-item hiding, letter-item visibility, clearing,
configured class reflection, and the return-widget contract. The editor target
and affected Widget, Inventory, Mailbox, Commission, and Story suites must pass.
