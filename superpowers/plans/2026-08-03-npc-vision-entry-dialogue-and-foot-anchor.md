# NPC Vision Entry Dialogue and Foot Anchor Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make NPC entry dialogue wait until the NPC can see a nearby player, while treating scene-anchor locations as character foot positions.

**Architecture:** Extend `FNPCSceneRow` with an entry-dialogue policy and perception parameters. `UNPCSceneSubsystem` owns a low-frequency wait state and starts dialogue only after distance, horizontal view-cone, and optional line-of-sight checks pass. Anchor-to-character conversion remains centralized in the scene subsystem and offsets teleport destinations by the NPC capsule half height.

**Tech Stack:** Unreal Engine 5.6 C++, `UWorldSubsystem`, StateTree tasks, AI navigation, collision queries, Unreal Automation Tests.

## Global Constraints

- Preserve all existing Blueprint dialogue UI links; Blueprints remain layout-only.
- Do not introduce a Behavior Tree or full NPC schedule system.
- `EntryDialogueTreeID` remains formal story dialogue; `InteractionDialogueTreeID` remains player-initiated stage dialogue.
- Normal scenes default to `OnSeePlayer`; forced cutscenes explicitly use `Immediate`.
- Anchor world location represents the NPC foot contact point; anchor Pitch and Roll never tilt a Character.
- Existing unrelated dirty worktree changes must not be overwritten, staged, or reverted.

---

### Task 1: Scene data and StateTree status contract

**Files:**
- Modify: `Source/Dispatch/Public/NPC/NPCSceneTypes.h`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Test: `Source/Dispatch/Private/Tests/NPCSceneTests.cpp`
- Test: `Source/Dispatch/Private/Tests/NPCSceneStateTreeTests.cpp`

**Interfaces:**
- Produces: `ENPCSceneEntryDialoguePolicy::{Immediate, OnSeePlayer, ManualInteraction}`.
- Produces: `FNPCSceneRow::EntryDialoguePolicy`, `EntryDialogueRadius`, `EntryDialogueHalfAngle`, `bRequireLineOfSight`.
- Produces: `ENPCSceneStatus::WaitingForPlayer` mapped to `EStateTreeRunStatus::Running`.

- [ ] **Step 1: Write failing reflection and status-mapping tests**

Add assertions that the four row properties exist, their defaults are `OnSeePlayer`, `500.0f`, `60.0f`, and `true`, and `WaitingForPlayer` maps to Running.

- [ ] **Step 2: Build DispatchEditor and verify RED**

Run the DispatchEditor build. Expected: compile failure because the enum, fields, and status do not exist.

- [ ] **Step 3: Add the minimal enum, fields, validation metadata, and status mapping**

Use `ClampMin="1.0"` for radius and `ClampMin="1.0", ClampMax="180.0"` for half-angle. Add `WaitingForPlayer` without changing the meaning of existing status values in code.

- [ ] **Step 4: Build and run `Dispatch.NPC.Scene` tests**

Expected: data and StateTree status tests pass; runtime tests may still fail only after later RED tests are added.

### Task 2: Foot-position anchor conversion

**Files:**
- Modify: `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h`
- Modify: `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp`
- Test: `Source/Dispatch/Private/Tests/NPCSceneRuntimeTests.cpp`
- Test: `Source/Dispatch/Private/Tests/NPCSceneOrchestrationTests.cpp`

**Interfaces:**
- Produces: `FVector GetCharacterActorLocationForAnchor(const ANPCCharacter&, const ANPCSceneAnchor&) const` or an equivalent private helper.
- Produces: anchor rotation conversion that returns `(Pitch=0, Yaw=AnchorYaw, Roll=0)`.

- [ ] **Step 1: Write a failing teleport foot-position test**

Spawn an NPC with a known scaled capsule half height and an anchor on `Z=125`. Play a Teleport scene and assert `NPC.ActorZ - ScaledCapsuleHalfHeight == Anchor.Z`, X/Y match, Yaw matches, and Pitch/Roll are zero.

- [ ] **Step 2: Run `Dispatch.NPC.Scene` tests and verify RED**

Expected: the actor center equals anchor Z under current code, so the foot-position assertion fails.

- [ ] **Step 3: Implement capsule offset and upright anchor rotation**

For Teleport, set actor location to `AnchorLocation + UpVector * GetScaledCapsuleHalfHeight()`. For arrival facing, copy only anchor Yaw. Keep AI Move destination at the anchor/nav-floor location and keep RestorePrevious using the original cached Actor transform.

- [ ] **Step 4: Run scene runtime tests and verify GREEN**

Expected: Teleport, Move completion, StayAtAnchor, and RestorePrevious tests pass.

### Task 3: Player visibility-gated entry dialogue

**Files:**
- Modify: `Source/Dispatch/Public/NPC/NPCSceneSubsystem.h`
- Modify: `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp`
- Test: `Source/Dispatch/Private/Tests/NPCSceneDialogueTests.cpp`
- Test: `Source/Dispatch/Private/Tests/NPCSceneOrchestrationTests.cpp`

**Interfaces:**
- Produces: a low-frequency `UNPCSceneSubsystem::Tick(float)` implementation through `UTickableWorldSubsystem` or an equivalent world timer owned by the subsystem.
- Produces: private `TryStartEntryDialogue`, `CanNPCSeePlayer`, and cleanup behavior for waiting scenes.
- Consumes: `FNPCSceneRow` policy, radius, half-angle, and LOS fields from Task 1.

- [ ] **Step 1: Write failing policy-path tests**

Cover: Immediate starts at arrival; ManualInteraction does not auto-start; OnSeePlayer reaches `WaitingForPlayer` without starting immediately.

- [ ] **Step 2: Write failing perception tests**

Use deterministic test hooks for player location and LOS when necessary. Assert no start when too far, behind the view cone, or blocked; assert exactly one start when within radius, inside the cone, and unblocked.

- [ ] **Step 3: Run `Dispatch.NPC.Scene` tests and verify RED**

Expected: current FinalizeArrival starts all configured Entry dialogue immediately and lacks `WaitingForPlayer`.

- [ ] **Step 4: Implement minimal policy routing**

Refactor FinalizeArrival so Immediate calls `TryStartEntryDialogue`, OnSeePlayer stores `WaitingForPlayer`, and ManualInteraction enters Active. Preserve `bWaitForEntryDialogue` after a successful start.

- [ ] **Step 5: Implement low-frequency perception checks**

Every 0.1 seconds, process only active records in `WaitingForPlayer`. Check squared distance first, horizontal view-cone dot product second, and `ECC_Visibility` line trace last. Ignore the NPC and player actors in the trace. On success, pause the routine, rotate to player Yaw, then start the existing story dialogue.

- [ ] **Step 6: Preserve failure and cleanup semantics**

An occupied dialogue keeps the scene waiting. Missing local player keeps it waiting. Destroyed NPC/anchor fails and cleans up. Invalid/start-failed Entry dialogue uses the existing rollback path. Scene replacement and StateTree abort remove the old waiting record.

- [ ] **Step 7: Run scene/dialogue tests and verify GREEN**

Expected: formal and interaction dialogue IDs remain distinct, waits are scoped to the correct NPC/dialogue, and routine restoration still passes.

### Task 4: Verification and editor handoff

**Files:**
- Modify only if failures reveal regressions in the files above.

**Interfaces:**
- Produces: a buildable DispatchEditor and passing relevant automation suites.

- [ ] **Step 1: Build DispatchEditor**

Run `Build.bat DispatchEditor Win64 Development -Project=... -WaitMutex -FromMsBuild`. Expected: exit code 0.

- [ ] **Step 2: Run focused automation suites**

Run `Dispatch.NPC`, then the available `Dispatch.Dialogue` and `Dispatch.StoryFlow` prefixes. Expected: all new tests pass; report any pre-existing unrelated failure separately with its exact test name.

- [ ] **Step 3: Inspect the final diff**

Confirm only intended C++/test files plus this plan changed during this implementation, and no user Blueprint, map, DataTable, or config asset was overwritten.

- [ ] **Step 4: Deliver editor configuration instructions**

Tell the user to set `EntryDialoguePolicy=OnSeePlayer`, radius/angle/LOS in `DT_NPCScenes`, and place the anchor root on the floor with its red X axis as the initial facing direction.
