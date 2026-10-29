# Prison Arrival Blink Transition Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a short C++ prison-arrival transition that follows the existing intro with blinking, a brief automated walk, a blackout cage transfer and sound, a guard subtitle, and a final daylight wake-up that restores gameplay.

**Architecture:** A native `UBlinkTransitionWidget` owns the reusable visual overlay and animation delegates. A placeable `APrisonArrivalSequenceActor` binds to the existing intro completion delegate and runs the scene as a deterministic state machine while reusing the existing player, NPC subtitle, HUD, and day/night systems.

**Tech Stack:** Unreal Engine 5.6, C++17, UMG/Slate, Unreal Automation Testing, existing Dispatch gameplay classes.

## Global Constraints

- Put all new runtime classes under `Story/PrisonArrival/`.
- Do not create a new NPC or guard class.
- Do not use NavMesh or Level Sequence for the escorted movement.
- Do not remove or recreate the existing player HUD.
- Do not push commits or test artifacts to GitHub.
- Switch to daylight with `SetDayNight(true)` without incrementing `CurrentDay`.

---

## File Map

- Create `Source/Dispatch/Public/Story/PrisonArrival/BlinkTransitionWidget.h`: native eyelid widget API and completion delegates.
- Create `Source/Dispatch/Private/Story/PrisonArrival/BlinkTransitionWidget.cpp`: UMG construction, painting, animation, and blur.
- Create `Source/Dispatch/Public/Story/PrisonArrival/PrisonArrivalSequenceActor.h`: editor configuration and state machine declaration.
- Create `Source/Dispatch/Private/Story/PrisonArrival/PrisonArrivalSequenceActor.cpp`: intro binding, movement, cage transfer, subtitle, daylight, HUD/input ownership, and cleanup.
- Create `Source/Dispatch/Private/Tests/PrisonArrivalSequenceTests.cpp`: local feature coverage only; no generated assets or reports.

### Task 1: Native blink overlay

**Files:**
- Create: `Source/Dispatch/Public/Story/PrisonArrival/BlinkTransitionWidget.h`
- Create: `Source/Dispatch/Private/Story/PrisonArrival/BlinkTransitionWidget.cpp`
- Create: `Source/Dispatch/Private/Tests/PrisonArrivalSequenceTests.cpp`

**Interfaces:**
- Produces: `OpenEyes(float Duration, float TargetOpenAmount)`, `CloseEyes(float Duration)`, `SetEyesOpenAmount(float Amount)`, `SetBlurStrength(float Strength)`, `IsFullyClosed()`, `IsFullyOpen()`, `OnEyesOpened`, and `OnEyesClosed`.

- [ ] **Step 1: Write failing API/default tests**

Add automation coverage that reflects the widget functions and delegates, constructs a widget with `NewObject`, verifies the default open amount is fully closed, and verifies zero-duration open/close calls finish synchronously.

- [ ] **Step 2: Run the focused test and verify RED**

Build the editor target. Expected: compile failure because `BlinkTransitionWidget.h` does not exist.

- [ ] **Step 3: Implement the minimal native widget**

Declare dynamic multicast completion delegates, clamp open amount to `[0,1]`, construct a full-screen `UBackgroundBlur` root, animate with `NativeTick`, invalidate paint on changes, and draw two black rectangles in `NativePaint`. Zero-duration transitions set the target and broadcast immediately.

- [ ] **Step 4: Build and run focused tests**

Build `DispatchEditor Win64 Development`, then run `Dispatch.Story.PrisonArrival.Blink`. Expected: build succeeds and all blink tests pass.

### Task 2: Prison arrival state machine

**Files:**
- Create: `Source/Dispatch/Public/Story/PrisonArrival/PrisonArrivalSequenceActor.h`
- Create: `Source/Dispatch/Private/Story/PrisonArrival/PrisonArrivalSequenceActor.cpp`
- Modify: `Source/Dispatch/Private/Tests/PrisonArrivalSequenceTests.cpp`

**Interfaces:**
- Consumes: all `UBlinkTransitionWidget` operations and completion delegates from Task 1.
- Produces: `StartSequence()`, `AbortSequence()`, `GetCurrentState()`, `IsSequencePlaying()`, and `OnSequenceFinished`.

- [ ] **Step 1: Write failing reflection/default tests**

Verify the actor exposes the required functions/delegate, defaults to `WaitingForIntro`, has ticking disabled before playback, uses non-negative default timings, and exposes marker/NPC/audio references as Blueprint-editable properties.

- [ ] **Step 2: Run focused tests and verify RED**

Build the editor target. Expected: compile failure because `PrisonArrivalSequenceActor.h` does not exist.

- [ ] **Step 3: Implement intro binding and state transitions**

Implement `WaitingForIntro -> OpeningForEscort -> EscortWalk -> ClosingForCage -> CageBlackout -> OpeningInCage -> GuardSubtitle -> ClosingForMorning -> MorningBlackout -> FinalOpening -> Finished`. Use blink delegates for animation boundaries and actor Tick only for movement/holds. Interpolate the player between walk markers without sweep. While fully black, move the player to `CageMarker`, orient the view through `BeginTeleportViewBlend`, and play `CageCloseSound`.

- [ ] **Step 4: Reuse existing systems**

Find and collapse the existing `UPlayerHUDWidget`, capture its visibility, pair `SetCinematicMode(true/false)`, capture and restore `APlayerCharacter::IsPlayerInputBlocked()`, call `GuardNPC->OnNPCSubtitleRequested` without entering conversation, and call `DayNightManager->SetDayNight(true)` only during `MorningBlackout`.

- [ ] **Step 5: Implement safe fallbacks**

Required player/controller/cage failures abort after restoring HUD/input and removing the overlay. Missing walk markers skip movement, missing guard skips subtitles, missing sound continues silently, missing day/night manager preserves time, and missing guard destination hides the guard. Repeated abort/completion and `EndPlay` are safe.

- [ ] **Step 6: Build and run focused tests**

Build `DispatchEditor Win64 Development`, then run `Dispatch.Story.PrisonArrival`. Expected: all focused tests pass.

### Task 3: Integration verification and handoff

**Files:**
- Modify only if verification finds a defect: files created by Tasks 1-2.

**Interfaces:**
- Consumes: completed blink overlay and prison-arrival actor.
- Produces: verified editor-facing setup instructions.

- [ ] **Step 1: Run static checks**

Run `git diff --check` and `git status --short`. Expected: no whitespace errors and no generated Unreal files, logs, or reports among changes.

- [ ] **Step 2: Build the editor target**

Run UE 5.6 `Build.bat` for `DispatchEditor Win64 Development`. Expected: `Result: Succeeded`.

- [ ] **Step 3: Run focused automation locally**

Run UnrealEditor-Cmd with `Automation RunTests Dispatch.Story.PrisonArrival;Quit`. Expected: all tests pass. Do not add produced logs to Git.

- [ ] **Step 4: Run an unattended editor smoke launch**

Launch UnrealEditor-Cmd with `-ExecCmds=Quit -unattended -nop4 -nullrhi -nosplash -log`. Expected: clean startup/shutdown without class load errors.

- [ ] **Step 5: Report Blueprint setup**

Tell the user to create a Blueprint child of `APrisonArrivalSequenceActor`, place it in the Demo map, and assign `IntroSequence`, `GuardNPC`, `WalkStartMarker`, `WalkEndMarker`, `CageMarker`, `CageCloseSound`, and timings. Do not push to GitHub.

