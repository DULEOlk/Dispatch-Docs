# Runtime and gameplay bridge review

Read-only review of the interrupted diff and current source. No repository files changed; no builds or tests run by this reviewer. Findings are from source tracing, with concrete reproduction paths below. Parent owns Campaign/date/RootStateTree integration and executor/dialogue review.

## Findings

### P2 — A valid OncePerInstance clue rule permanently blocks the event queue

Location: `Source/Dispatch/Private/Story/Narrative/StoryGameplayBridgeComponent.cpp:164` (also runtime lines 367–368, bridge lines 285–290).

`HandleClue` always supplies an invalid `FGuid()` as InstanceID. Episode validation permits `OncePerInstance` for a ClueCollected EventKey, but SubmitStoryEvent rejects any matching rule with that repeat policy and missing InstanceID. Reproduction: use the existing GameplayBridge.Clue fixture, set Trigger.Repeat to OncePerInstance, start, collect the awaited clue, flush. Flush returns false, the fact/reward never commits, and WaitForClue cannot complete because the rejected event remains at the queue head. Every subsequent gameplay event is blocked behind it. Provide a stable discovery identity, or reject this authoring combination before campaign start with an explicit diagnostic.

### P2 — Reentrant commission commits are forwarded in reverse chronological order

Location: `Source/Dispatch/Private/Story/Narrative/StoryGameplayBridgeComponent.cpp:203–207` (producer broadcasts at CommissionSubsystem.cpp:589,653).

The live bridge throws away StateChangeSerial and queues broadcasts in callback order. A committed listener may legally perform another commission operation before the bridge receives the outer notification. Reproduction: start the bridge, then add an OnStateCommittedNative observer which calls MarkCommissionReady(Instance.InstanceID) when it receives Active; confirm a real dispatch and flush. UE5.6 multicast delegates execute the later-added observer first (verified in local Core/Public/Delegates/MulticastDelegateBase.h). The nested ReadyToResolve broadcast reaches the bridge first; outer Active reaches it second. The subsystem ends ReadyToResolve, while the narrative ledger ends Active. A ReadyToResolve observer which calls ResolveCommission similarly leaves the ledger ReadyToResolve after actual resolution. Both trigger ordering and subsequent branch conditions are incorrect. Startup sorts by serial, so restart masks this bug. Serialize producer notifications or preserve and order live committed serials without discarding occurrences.

### P2 — Historical writing snapshots can be attributed to an unrelated day asset

Location: `Source/Dispatch/Private/Story/Narrative/StoryGameplayBridgeComponent.cpp:76–92`.

Live submissions carry the exact UDayDefinition, but FSubmittedDailyRecord stores only an integer Day. Startup resolves that integer against the new campaign catalog and manufactures WritingOption/WritingCredibility facts for whichever sole asset has that number. Reproduction: submit day asset A with DayNumber=1; stop; start a campaign whose sole day-1 catalog asset is B, with different options. Startup emits A's selected option IDs under B's SourceAsset (even IDs B does not define). A single candidate in the new catalog does not establish the original source. This can incorrectly satisfy branch/entry conditions despite snapshots avoiding occurrence rewards. Persist original source identity in the submission record and filter snapshots by it.

## Additional limitations / test gaps

- Commission facts are stamped with ResolveDay at observation/startup (`BuildCommissionFacts`, line 191). FCommissionInstance stores serial but no occurrence day. Restart on day 3 changes a day-1 outcome snapshot's DayNumber to 3; a listener advancing day before bridge notification similarly misdates live commits. Day coordination is assigned to the parent, but producer records still need original context for faithful snapshots.
- Only dialogue source events preserve producer EventID. Commission, writing, inference and clue notifications receive new EventIDs on each forwarding; writing/inference also receive new InstanceIDs. Runtime retry of an already queued event is stable, but replay of the same source notification is not deduplicated (EveryEvent rewards repeat; writing/inference OncePerInstance rewards can repeat). Current producers suppress ordinary duplicate operations; tests should distinguish operation idempotency from source replay identity.
- Writing submission is expanded into section facts followed by credibility. A trigger on the first section conditioned on credibility or a later section sees an incomplete submission snapshot. Commission resolved-state triggers conditioned on the new outcome have the analogous ordering dependency. Clarify whether multi-fact committed operations must be atomic at rule-observation level; existing tests only use independent unconditional rewards.
- A new event while RunState is EpisodeCompleted is queued, rejected on flush and disables bridge ticking. Starting another episode does not reenable it; only an explicit successful FlushGameplayEvents does. Episode transition integration should explicitly cover this queue lifecycle, including a next WaitForClue action.
- Existing trigger tests cover repeat policies, event retry deduplication, priority conflict atomicity, effect overflow rollback, pre-effect snapshot isolation, and repeated trigger receipt restore. Existing bridge tests cover real producers, successful snapshots, clue waits, startup failure cleanup and competing bridge ownership. They do not cover the three findings above, source callback reordering, original source/day preservation, or transient completed-episode acceptance/recovery.

## Completed implementation in this scope

- Trigger data, episode/campaign reference validation, event and rule receipts, runtime-owned fact protection, and staged all-or-nothing single-event effects are implemented.
- Initial gameplay snapshots are seeded before entry evaluation and do not fire occurrence rules.
- Bridge subscriptions and stop cleanup exist for clues, commissions, writing, inference and dialogue; rejected queued events retain identity/order for explicit retry.
- Writing/inference notifications are emitted after committing records and carry local stable payload copies; commission notifications follow committed lifecycle changes and do not expose the hidden outcome before applied resolution.
- Commission startup folding sorts committed instances by serial and snapshots fold to latest value per key.
