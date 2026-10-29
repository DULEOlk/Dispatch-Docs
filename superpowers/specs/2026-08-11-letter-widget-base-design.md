# Letter Widget C++ Base Design

## Goal

Move the mailbox letter presentation and close workflow into C++ so the existing
`WBP_LetterUI` only owns its visual layout.

## Widget Base

Add `ULetterWidgetBase`, derived from `UUserWidget`. It binds the existing
optional widgets named `Text_Name`, `Text_Description`, and `Button_Close`.
Initialization receives a `ULetterDefinition`, the source `AMailbox`, and the
reading `APlayerCharacter`. It displays the letter title and joins the readable
`ContentSegments` into the body, falling back to `InventorySummary` when the
letter contains no body segments.

The base class owns UI input mode while constructed. Clicking `Button_Close`
calls a public `CloseLetter` function. For mailbox-opened letters, closing first
calls `AMailbox::CompleteLetterReading`; if completion fails, the widget remains
open so the letter cannot be silently lost. Successful close removes the widget,
restores game input, and hides the mouse cursor. When the source mailbox is null,
the same widget can later be reused to read an already-collected inventory letter
without collecting it again.

## Mailbox Integration

Add a configurable `LetterWidgetClass` to `AMailbox`. When the player interacts
with a directly delivered `ULetterDefinition`, the mailbox creates the configured
widget, initializes it, and adds it to the viewport. If no widget class is
configured or creation fails, the existing `OnLetterOpened` Blueprint event
remains as a compatibility fallback.

## Blueprint Setup

After compilation:

1. Reparent `WBP_LetterUI` to `LetterWidgetBase`.
2. Set `BP_Mailbox.LetterWidgetClass` to `WBP_LetterUI`.
3. Remove the old `SetLetterInfo` and `Button_Close` event graph logic after the
   reparented widget compiles successfully.

## Compatibility and Safety

The existing legacy `OnLetterTaken` and `LetterID`/data-table delivery path stays
unchanged. Existing Blueprint assets are not edited by Codex. All widget bindings
are optional so a temporarily incomplete Blueprint does not prevent compilation.

## Verification

Automation tests cover body formatting, initialization validation, mailbox widget
configuration exposure, close-time collection, and legacy API compatibility.
The editor target plus Inventory, Mailbox, Commission, and Story suites must pass.
