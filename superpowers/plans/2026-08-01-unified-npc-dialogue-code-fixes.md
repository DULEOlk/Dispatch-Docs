# Unified NPC Dialogue Code Fixes Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the unified NPC dialogue and dynamic interaction C++ implementation compile and provide a complete, tested session/UI lifecycle without modifying user-authored content assets.

**Architecture:** `UDialogueSubsystem` remains the model/session owner. `UDialogueUIControllerSubsystem` binds at LocalPlayer initialization and owns one `UDialogueScreenWidget`; the screen and entry C++ classes forward input without NPC Blueprint logic. Story sessions and player-interaction sessions use distinct close semantics and one preserved NPC state snapshot.

**Tech Stack:** Unreal Engine 5.6 C++, UGameInstanceSubsystem, ULocalPlayerSubsystem, UMG/ListView, Unreal Automation Tests.

## Global Constraints

- Do not edit or delete user-authored `.uasset` files, maps, StateTrees, or DataTables.
- Widget Blueprints remain layout-only; all runtime behavior belongs in C++.
- NPC proactive dialogue remains story-controlled through StateTree/NPC Scene.
- Preserve compatibility entry points until content migration is complete.
- Write a failing automation test before each production behavior fix.

---

### Task 1: Restore compilation and delegate contracts

**Files:**
- Modify: `Source/Dispatch/Public/Quest/DialogueSubsystem.h`
- Modify: `Source/Dispatch/Public/Widget/DialogueUIControllerSubsystem.h`
- Modify: `Source/Dispatch/Public/Widget/DialogueScreenWidget.h`
- Modify: `Source/Dispatch/Public/Widget/DialogueOptionViewModel.h`
- Modify: `Source/Dispatch/Private/Widget/DialogueScreenWidget.cpp`
- Modify: `Source/Dispatch/Private/Widget/DialogueUIControllerSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCDialogueTests.cpp`

**Interfaces:**
- Produces native node/end delegates with exact signatures usable through `AddUObject`.
- Preserves Blueprint multicast delegates for existing Blueprint consumers.

- [ ] Add native `FOnDialogueNodeStartedNative(FName, const FDialogueNodeRow&)` and `FOnDialogueEndedWithIDNative(FName)` delegates and broadcast them beside existing delegates.
- [ ] Bind C++ UI classes to native delegates only; keep `FDelegateHandle` removal symmetric.
- [ ] Include/forward-declare `EDialogueSessionMode` and UObject classes consistently.
- [ ] Update legacy test choice construction to assign fields explicitly.
- [ ] Compile `DispatchEditor`; expected result is successful compilation or only later behavioral-test failures.

### Task 2: Make session lifecycle deterministic

**Files:**
- Modify: `Source/Dispatch/Private/Tests/NPCDialogueTests.cpp`
- Modify: `Source/Dispatch/Public/Quest/DialogueSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/DialogueSubsystem.cpp`

**Interfaces:**
- Produces `bool IsDialogueSessionActive() const`.
- Produces a forced close path distinct from return-to-menu dialogue completion.

- [ ] Add failing tests proving Exit closes a menu, starting an inquiry preserves the original NPC snapshot, and Story takeover closes a menu cleanly.
- [ ] Run focused Dialogue tests and confirm the new assertions fail for the observed lifecycle reasons.
- [ ] Track session activity independently from `ActiveDialogueID`; snapshot NPC state once at session start.
- [ ] Split “finish current inquiry” from “close whole session”; Exit always force-closes.
- [ ] Ensure interaction inquiry completion returns to the menu only when configured and final close restores the original behavior/routine.
- [ ] Run focused tests and confirm all lifecycle tests pass.

### Task 3: Complete C++ UI creation and input forwarding

**Files:**
- Create: `Source/Dispatch/Private/Tests/DialogueUIControllerTests.cpp`
- Modify: `Source/Dispatch/Public/Widget/DialogueUIControllerSubsystem.h`
- Modify: `Source/Dispatch/Private/Widget/DialogueUIControllerSubsystem.cpp`
- Modify: `Source/Dispatch/Public/Widget/DialogueScreenWidget.h`
- Modify: `Source/Dispatch/Private/Widget/DialogueScreenWidget.cpp`
- Modify: `Source/Dispatch/Public/Widget/DialogueOptionEntryWidget.h`
- Modify: `Source/Dispatch/Private/Widget/DialogueOptionEntryWidget.cpp`

**Interfaces:**
- UI controller binds during `Initialize` and synchronizes the current node/menu after widget creation.
- Entry widget exposes a native selection delegate carrying `UDialogueOptionViewModel*`.
- Screen handles ListView selection and keyboard/mouse continue in C++.

- [ ] Add failing reflection/unit tests for initialization binding hooks, option forwarding API, and continue/close operations.
- [ ] Bind UI controller to DialogueSubsystem during initialization, not after widget creation.
- [ ] After creation, render the current line/choices/menu immediately so the first node cannot be missed.
- [ ] Bind ListView item click and entry button click to one C++ selection path.
- [ ] Add a C++ continue button binding and `NativeOnKeyDown` support for confirm/cancel input.
- [ ] Run UI-focused tests and compile.

### Task 4: Connect Story and special actions

**Files:**
- Modify: `Source/Dispatch/Private/Tests/NPCSceneOrchestrationTests.cpp`
- Modify: `Source/Dispatch/Private/NPC/NPCSceneSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Story/Flow/StoryStateTreeNodes.cpp`
- Modify: `Source/Dispatch/Public/Quest/DialogueSubsystem.h`
- Modify: `Source/Dispatch/Private/Quest/DialogueSubsystem.cpp`

**Interfaces:**
- NPC Scene and Story StateTree use `StartStoryDialogueForNPC`.
- Dialogue subsystem emits `OnSpecialInteractionRequested(ENPCInteractionActionType, ANPCCharacter*)`.

- [ ] Add failing tests that scene entry dialogue is explicitly Story mode and special actions emit the correct typed request.
- [ ] Change NPC Scene and Story dialogue task to the Story entry point.
- [ ] Add and broadcast the typed special-action delegate for NPC info/commission.
- [ ] Hide OpenCommission unless the NPC is eligible and a handler is bound.
- [ ] Run focused NPC Scene and Dialogue tests.

### Task 5: Validation, regression, and handoff

**Files:**
- Modify: `Source/Dispatch/Private/Tests/NPCDialogueTests.cpp`
- Modify: `Source/Dispatch/Private/Tests/DialogueUIControllerTests.cpp`

**Interfaces:**
- Validation rejects duplicate/empty visible option IDs and invalid dialogue targets with actionable logs.

- [ ] Add failing validation tests for missing interaction tables/rows, duplicate OptionID/ChoiceID, and missing targets.
- [ ] Implement minimal validation without changing content assets.
- [ ] Build `DispatchEditor`.
- [ ] Run `Dispatch.Dialogue`, `Dispatch.NPC.Scene`, `Dispatch.Story.Flow`, and full `Dispatch.NPC`.
- [ ] Run `git diff --check` and confirm only intended C++/test/plan files were edited by this repair.
