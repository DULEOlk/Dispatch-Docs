# NPC Interaction Range and Exit UI Design

## Scope

Adjust the existing NPC player-interaction presentation only. Do not change Story dialogue completion, scene ownership, dialogue data, or Blueprint logic.

## Interaction range

- Change the default `ANPCCharacter::InteractionBox` horizontal extent from 140 cm to 250 cm.
- Keep the existing vertical extent and Pawn-overlap collision behavior.
- Visibility tracing still has priority when it identifies an interactable; the overlap candidate remains the fallback.

## Exit controls

- The interaction menu keeps its generated `Exit` option, displayed as `离开`.
- `Button_Close` is collapsed while the player is viewing the interaction menu, avoiding duplicate `离开` and `关闭` controls.
- During a dialogue tree launched from the interaction menu, `Button_Close` is visible and returns to the interaction menu according to the existing player-interaction session behavior.
- Story dialogue continues to hide `Button_Close` and ignore player close requests.

## Verification

- Extend the NPC interaction test to require a 250 cm horizontal interaction extent.
- Add a testable close-button visibility policy covering Story, interaction-menu, and interaction-dialogue states.
- Build `DispatchEditor` and run the relevant NPC interaction and dialogue UI automation tests.
