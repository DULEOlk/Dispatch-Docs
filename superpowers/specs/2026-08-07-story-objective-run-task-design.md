# Story Objective Run Task Design

## Scope

Replace the repeated StateTree sequence `Set Objective -> Show Marker -> Wait For Signal -> Hide Marker -> Clear Objective` with one reusable, data-driven task. This phase changes C++ only. Existing StateTree nodes and binary assets remain valid and unchanged.

Letter DataAsset migration, letter-UI close signaling, document migration, and NPC scene/dialogue migration are separate follow-up phases.

## Chosen approach

Create a `UStoryObjectiveDefinition` primary data asset and a long-running `Story: Run Objective` StateTree task. This is preferred over embedding a large struct in every StateTree because it removes repeated configuration, and preferred over a new objective subsystem because `AStoryDirector` already owns the active objective presentation and story-signal history.

## Objective definition

`UStoryObjectiveDefinition` contains:

- stable non-empty `ObjectiveID`;
- non-empty multiline `ObjectiveText`;
- optional compass guidance through `bShowMarker` and `TargetID`;
- completion filter through required `CompletionTrigger`, optional `CompletionSourceID`, optional `CompletionContextName`, and optional integer context matching.

The asset exposes pure validation and signal-matching functions so configuration and matching behavior can be covered without constructing a StateTree execution context.

## Runtime ownership

`AStoryDirector` gains `CurrentObjectiveID`, `BeginObjective`, and `EndObjective`.

`BeginObjective` validates the definition, stores its ID, displays its text, and either sets or clears the marker. `EndObjective` only clears presentation when the supplied ID still owns the active objective. This prevents an exiting stale task from clearing a newer objective.

The existing `SetCurrentObjective`, `ClearCurrentObjective`, `SetObjectiveMarkerTarget`, and `ClearObjectiveMarkerTarget` APIs remain available for existing assets.

## StateTree task lifecycle

`FStoryStateTreeRunObjectiveTask` owns a direct `UStoryObjectiveDefinition` reference.

- `EnterState`: validate the director and asset, record the current signal serial, call `BeginObjective`, and return `Running`.
- `Tick`: call the existing `HasMatchingSignalSince` with the asset's completion filter; return `Succeeded` only for a newer matching signal.
- `ExitState`: if this task began the objective, call `EndObjective` with the definition's ID on success, failure, or interruption.

The task does not resolve the compass target itself. The existing compass and navigation subsystem continue to hide unresolved targets safely and track registered actors dynamically.

## Compatibility and errors

- Existing granular objective tasks are retained.
- Invalid definitions fail on entry without changing the director.
- A marker-enabled definition without a `TargetID` is invalid.
- A marker-disabled definition clears any previous objective marker when it begins.
- Signals submitted before task entry do not complete the new objective.

## Verification

Add automation coverage for definition validation, optional filter matching, director ownership, and reflected StateTree task API. Verify the new tests fail before implementation, then build `DispatchEditor` and run focused `Dispatch.Story.Objective`, `Dispatch.Story.HUDStatus`, and `Dispatch.Story.SignalWait` tests.
