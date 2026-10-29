# Prison Arrival Blink Transition Design

## Goal

Connect the existing illustrated intro to first-person gameplay with a short prison-arrival sequence. The player sees a brief escorted walk, is moved into the cage while their eyes are closed, hears the cage close, wakes to a short guard subtitle, closes their eyes again, and finally wakes in daylight with normal control.

The sequence is a non-interactive cinematic transition. It must reuse the existing player, NPC, subtitle, HUD, intro, and day/night systems.

## Scope

The first version provides:

- a reusable C++ eyelid overlay with open, close, and temporary blur;
- a C++ prison-arrival state machine;
- deterministic movement between editor-assigned markers without NavMesh;
- a blackout-safe teleport into the cage;
- cage-door audio played only after the screen is fully closed;
- one automatic guard subtitle using the existing NPC presentation;
- a closed-eye transition to the next daylight state;
- HUD and input restoration only after the final eye opening.

The first version does not provide branching dialogue, player input during the sequence, a new guard class, Level Sequence integration, navigation-driven escort behavior, save-game persistence, or bespoke eyelid art.

## File Organization

All new runtime classes are grouped under a dedicated feature directory:

```text
Source/Dispatch/Public/Story/PrisonArrival/
    BlinkTransitionWidget.h
    PrisonArrivalSequenceActor.h

Source/Dispatch/Private/Story/PrisonArrival/
    BlinkTransitionWidget.cpp
    PrisonArrivalSequenceActor.cpp
```

Feature-specific automation coverage, if needed during implementation, belongs in the existing `Source/Dispatch/Private/Tests` directory and must not introduce generated reports, logs, or test assets. No changes will be pushed to a remote repository.

## Existing Systems Reused

- `AIntroSequenceActor::OnIntroFinished` starts the arrival sequence.
- `ANPCCharacter` remains the guard type; no subclass is added.
- `ANPCCharacter::OnNPCSubtitleRequested` uses the existing NPC subtitle Blueprint presentation.
- `UPlayerHUDWidget` is hidden and restored rather than removed or recreated.
- `ADayNightManager::SetDayNight(true)` switches the opening scene to the first playable daylight without incrementing the day counter.
- `APlayerController::SetCinematicMode` and `APlayerCharacter::SetBlockPlayerInput` prevent control during the transition.

## Blink Overlay

`UBlinkTransitionWidget` is a native `UUserWidget`. It constructs its Slate content in C++, so the feature does not require a new Widget Blueprint.

It owns two black eyelid panels that move from the top and bottom edges toward the center. An open amount of `0.0` means fully closed and `1.0` means fully open. A full-screen background blur is strongest during the first opening and fades as the player regains awareness.

Public operations:

- `OpenEyes(Duration, TargetOpenAmount)`
- `CloseEyes(Duration)`
- `SetEyesOpenAmount(Amount)`
- `SetBlurStrength(Strength)`
- `IsFullyClosed()`
- `IsFullyOpen()`

Completion delegates notify the sequence actor when opening or closing actually finishes. Zero and negative durations complete immediately, allowing safe editor iteration and deterministic tests.

The overlay is added above existing HUD widgets. It is non-interactive and never changes the player's input mode.

## Prison Arrival Sequence

`APrisonArrivalSequenceActor` is a placeable, Blueprintable C++ actor. Designers configure references and timings on its Blueprint instance.

Editor-assigned references:

- `IntroSequence`
- `GuardNPC`
- `WalkStartMarker`
- `WalkEndMarker`
- `CageMarker`
- optional `GuardAfterSequenceMarker`
- optional `DayNightManager`; when omitted, the actor locates the level instance
- `CageCloseSound`
- guard subtitle text

Editor-assigned timings:

- first open duration;
- escorted-walk duration;
- cage close duration;
- closed hold around the cage sound;
- cage open duration;
- subtitle hold duration;
- final close duration;
- final open duration.

### State Flow

```text
WaitingForIntro
  -> OpeningForEscort
  -> EscortWalk
  -> ClosingForCage
  -> CageBlackout
  -> OpeningInCage
  -> GuardSubtitle
  -> ClosingForMorning
  -> MorningBlackout
  -> FinalOpening
  -> Finished
```

At startup, the sequence binds to the configured intro actor. The intro continues to own its existing slideshow behavior. When `OnIntroFinished` fires, the arrival actor immediately restores cinematic blocking, hides the HUD, creates the closed eyelid overlay, and places the player at `WalkStartMarker` while the screen is black.

During `EscortWalk`, C++ interpolates the player actor from `WalkStartMarker` to `WalkEndMarker`. This is intentionally independent of NavMesh and AI. Collision sweep is disabled because the path is a designer-authored cinematic path and must finish deterministically. The player remains unable to move or look.

After `ClosingForCage` reaches fully closed, the player is placed at `CageMarker`. Only then is `CageCloseSound` played. This guarantees that teleporting and any cage-state changes are invisible.

After `OpeningInCage`, the guard's existing subtitle event is invoked once. No conversation state, choices, or dialogue input are enabled. The state advances automatically after the configured subtitle hold.

After the final close reaches fully black, `SetDayNight(true)` is called. This preserves the configured day number and only changes the time of day. The guard is hidden or moved to `GuardAfterSequenceMarker`, depending on configuration. The final opening removes the blur, restores the HUD visibility captured at sequence start, releases both input locks, removes the overlay, broadcasts completion, and disables ticking.

## HUD and Input Ownership

The arrival actor locates the existing `UPlayerHUDWidget` after the intro completes, records its current visibility, and changes it to `Collapsed`. It does not access the Blueprint-only `PlayerHUDRef`, remove the widget, or create a second HUD.

The actor stores whether movement and look input were already ignored before it started. On completion it restores those previous states rather than assuming they were enabled. It also stores and restores `APlayerCharacter::IsPlayerInputBlocked()`.

Cleanup is idempotent. Ending the sequence twice must not duplicate delegates, restore input incorrectly, or broadcast completion twice.

## Failure Handling

Required references are validated before the sequence starts.

- Missing intro reference: log once and remain inactive.
- Missing player controller, player character, or cage marker: abort safely, remove the overlay, and restore captured HUD/input state.
- Missing walk markers: skip the escorted interpolation and proceed to the cage blackout.
- Missing guard: skip the subtitle and continue.
- Missing cage sound: continue silently.
- Missing day/night manager: finish the visual sequence without changing time.
- Missing optional guard destination: hide the guard during the morning blackout.

No failure may leave the player behind a permanent black overlay or with input permanently blocked.

## Verification

Implementation verification will cover:

- compilation of `DispatchEditor Win64 Development`;
- state transitions for normal and zero-duration timings;
- exact ordering of full closure, cage teleport, and door audio request;
- missing-reference fallbacks;
- HUD visibility restoration;
- input restoration;
- day transition occurring only during the final blackout;
- one end-to-end Unreal automation run for the feature where practical.

Verification is local only. No remote push is part of this work, and generated test reports or temporary files are not added to source control.
