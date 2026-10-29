# Fixed Story Compass Guidance Design

## Scope

Add optional compass guidance for fixed story locations. Story StateTrees explicitly decide when guidance appears and disappears. Moving NPC guidance is out of scope.

The feature also adds a reusable way for a StateTree to stop until gameplay submits a matching story signal. This replaces automatic time-based progression for objectives that require player action.

## Responsibilities

### Compass widget

Create a `UCompassWidget` C++ base class and reparent `WBP_Compass` to it. The widget owns runtime calculations:

- read the owning player's control yaw;
- position and hide the North, East, South, and West labels;
- resolve the active fixed story target;
- position and hide the objective marker;
- fail safely when the player or target is unavailable.

UMG continues to own the compass frame, labels, marker image, anchors, sizing, colors, and animation styling. The existing Blueprint tick and `UpdateCompassText` calculation are removed after reparenting.

### Fixed navigation targets

Create a reusable actor component for fixed locations. A placed actor exposes a stable `TargetID`, such as `Location.Mailbox.Main`, and registers itself at runtime. A world-scoped registry resolves an active ID to an actor location without hard-referencing a level actor from a StateTree asset.

Duplicate non-empty IDs are configuration errors. Resolution failure hides the marker and logs a useful warning without blocking story flow.

### Story ownership

`AStoryDirector` stores the currently active compass target ID. This makes guidance state independent of widget construction order and allows a newly constructed HUD to restore the current marker.

Add two generic StateTree tasks:

- `Story: Show Objective Marker`, with a `TargetID` parameter;
- `Story: Hide Objective Marker`.

These tasks change presentation only. They do not decide whether an objective is complete.

### Waiting for gameplay

Add `Story: Wait For Signal`, a long-running StateTree task that subscribes to `AStoryDirector::OnStorySignalSubmitted` while active. It completes only when a submitted `FStorySignal` matches:

- required `Trigger`;
- optional `SourceID`;
- optional `ContextName`;
- optional `ContextValue` filtering only when explicitly enabled.

The task removes its subscription when the state exits. Unrelated signals leave it running.

## Mailbox integration

Opening a delivered letter UI is the completion point for the first mailbox objective. The mailbox module submits one generic signal for every opened letter:

- Trigger: `Story.Trigger.MailOpened`;
- SourceID: `Mailbox`;
- ContextName: the opened Letter ID.

No first-letter-specific Blueprint function or branch is added. `ST_Day01` distinguishes the introductory letter by configuring `ContextName = Letter_Day01_Intro` on `Story: Wait For Signal`.

The intended Day 1 sequence is:

1. Deliver the introductory letter.
2. Set the objective text.
3. Show the marker for `Location.Mailbox.Main`.
4. Wait for `MailOpened` with the introductory Letter ID.
5. Hide the marker.
6. Clear or replace the objective and continue.

The reusable C++ mailbox path should submit the signal at the point where the letter-open action succeeds. Blueprint remains responsible for constructing and styling `WBP_LetterUI` until that UI flow is separately migrated.

## Day definition assets

`DA_Day01` through `DA_Day07` remain unchanged. `DayNumber` is required by `DayFlowManager`; `RequiredWorkCount` only gates ending a day. `AvailableEvents` is currently unused and is not part of this feature.

## Verification

Add focused automation coverage for:

- compass node and task metadata;
- active marker persistence on `AStoryDirector`;
- fixed target registration and lookup, including duplicate or missing IDs;
- signal matching with trigger and optional context filters;
- unrelated signals not completing the wait;
- mailbox open signals carrying the delivered Letter ID;
- compass angle mapping and visibility calculations as pure testable helpers.

Run an incremental `DispatchEditor` build, then only the relevant Dispatch story, mailbox, and compass automation tests. Do not run the full suite unless focused verification exposes a broader dependency.

## Out of scope

- NPC or other moving-target guidance;
- automatic hints based on elapsed time or player distance;
- distance-based marker dismissal;
- changes to `DA_Day01` event lists;
- redesigning compass artwork or letter UI visuals.
