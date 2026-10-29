# Non-cancelable Story Dialogue Implementation Plan

1. Add a regression test proving player close requests are rejected during Story sessions and accepted during PlayerInteraction sessions.
2. Add `CanPlayerCloseCurrentSession` and `RequestCloseDialogueSession` to `UDialogueSubsystem` without changing the internal/system `CloseDialogueSession` path.
3. Route dialogue UI close actions through the player request API, hide the close button for Story sessions, and consume Story-session close keys without ending the session.
4. Build `DispatchEditor` and run `Dispatch.Dialogue`, `Dispatch.NPC.Scene`, and `Dispatch.Story.Flow` automation tests.
