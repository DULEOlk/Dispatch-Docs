Read-only review: action execution and dialogue events

Scope: StoryActionExecutorComponent, DialogueSubsystem event additions, LetterWidgetBase, PlayerCharacter writing presentation, and related tests. No source edits, builds, or tests performed; parent owns validation.

Concrete findings

1. [P2] Writing cleanup can close a replacement session at the same desk. Source/Dispatch/Private/Story/Narrative/StoryActionExecutorComponent.cpp:214 (related 170-173). Reproduction: start the executor OpenWriting request; call Player->ExitWritingMode(), then Player->TryEnterWritingMode(the same desk) before the executor processes its deliberately delayed result. The exit callback queues failure but retains bOwnsWriting. ClearOwnedPresentation checks only desk equality and exits the replacement session, removing its widget and resetting its input. A request/session token or captured owned widget is needed, and ownership should end on the original exit. This is a concrete unrelated-session cleanup defect, not merely missing test coverage.

2. [P2] OpenCommission starts over an existing inspect presentation. Source/Dispatch/Private/Story/Narrative/StoryActionExecutorComponent.cpp:151-153. Reproduction: enter notice-board inspect mode and issue an OpenCommission action. This guard checks writing and an active dialogue subsystem session, but permits IsInspecting(). OpenStoryCommission then enters NPC dialogue mode while the inspect mode remains active. Closing/cancelling the commission subsequently calls ExitNPCDialogueMode(), which sets GameOnly input and hides the cursor even though the notice-board inspection is still active (PlayerCharacter.cpp:729-754); alternatively Escape exits inspection underneath the commission and sets GameOnly input there. PresentLetter and StartDialogue already reject inspection. Add the same presentation exclusivity check for this action, or explicitly coordinate/restores the prior mode.

Completed behavior observed

- Native executor subscription, request IDs, deferred results, stale-request checks, and owned dialogue session identity.
- PresentLetter closes only after actual mailbox reading/collection; LetterWidgetBase reports explicit success and destruction cancellation.
- StartDialogue handles synchronous terminal nodes, result identity, cancellation/failure, and lost widgets/session detection.
- OpenCommission listens for actual dispatch/cancellation; OpenWriting listens for committed daily submission/exit; RequestScene waits for Active status.
- Executor crosses a frame boundary and calls the gameplay bridge flush before CompleteAction; queued facts can therefore precede completion branch selection in the intended director composition.
- Dialogue events contain fresh event IDs, per-start session IDs, authored source/node/choice identities, and Completed/Cancelled/Failed results. The new deferral puts a successful choice before its synchronous FinishDialogue result.
- TryEnterWritingMode refuses missing controller/widget without leaving bIsWriting set, retains the active widget, and ExitWritingMode removes it.

Remaining verification/coverage (not established bugs)

- StoryActionExecutorTests.cpp currently tests binding/unbinding, unavailable writing, and LetterWidgetBase close idempotency only. It does not drive an actual executor request through any successful gameplay action and resulting branch. Add integration coverage for all five presentation actions, cancellation, missing targets/widgets, stop/restart cleanup, stale callbacks, and replacement sessions. Tests that merely manually CompleteAction do not verify this wiring.
- StoryGameplayBridgeEventsTests.cpp tests live choices, cancellation, repeated session IDs, forwarding deduplication and restart snapshots. It does not test choice-to-Exit/FinishDialogue order or executor completion after those facts are committed.
- Cross-producer ordering needs an explicit test/design decision: choice destination effects run before the DialogueChoice publication (DialogueSubsystem.cpp:463-476); GrantClue publishes its clue delegate immediately, outside the dialogue-only deferred queue. Consequently the bridge receives a clue granted by the selected branch before the choice that caused it. A ClueCollected trigger conditioned on the new DialogueChoice sees no new choice. This is a concrete ordering consequence, but whether it violates the authored trigger contract should be confirmed before categorizing it as a required fix.
- Cancelling/removing an executor letter leaves its delivered CurrentLetter/bHasLetter in the mailbox. A subsequent PresentLetter request rejects that mailbox until the outstanding letter is consumed. Confirm desired cancellation/restart policy; no rollback is implemented.
- No actual viewport/UI behavior or Blueprint writing initialization has been exercised by this review.

Validation provenance: The two findings above are statically traced reproduction sequences, not executed runtime reproductions. Parent independently reports Development build success; Narrative + Story.Sources 29/30 passing, with ActionExecutor.Binding failing because NewObject<UNarrativeRuntimeSubsystem>() at StoryActionExecutorTests.cpp:17 omits its required GameInstance outer; related gameplay regressions 56/56 pass. This test fixture failure is separate from the production defects above.
