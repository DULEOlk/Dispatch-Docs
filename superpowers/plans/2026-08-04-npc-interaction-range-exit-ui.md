# NPC Interaction Range and Exit UI Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Increase the default NPC interaction range and remove the duplicate close control from the interaction menu.

**Architecture:** Keep the existing NPC overlap/trace interaction architecture. Increase only the NPC overlap extent, and centralize the close-button presentation rule in `UDialogueScreenWidget` so Story, interaction-menu, and interaction-dialogue states are distinguishable and testable.

**Tech Stack:** Unreal Engine 5.6, C++, UMG, Unreal Automation Framework.

## Global Constraints

- Preserve the generated `Exit` interaction option and its `离开` label.
- Do not modify Blueprint assets or Story dialogue completion behavior.
- Preserve user changes in the dirty worktree and do not commit production code without an explicit request.

---

### Task 1: Increase NPC interaction range

**Files:**
- Modify: `Source/Dispatch/Private/Tests/NPCSystemTests.cpp`
- Modify: `Source/Dispatch/Private/Characters/NPCCharacter.cpp`

**Interfaces:**
- Consumes: `ANPCCharacter::InteractionBox`
- Produces: a default unscaled horizontal box extent of 250 cm

- [ ] Add an assertion that a spawned NPC has `InteractionBox->GetUnscaledBoxExtent().X == 250.0f` and Y equals 250.0f.
- [ ] Run `Dispatch.NPC.Character.DialogueInteractPrompt` and confirm the new assertion fails with the current 140 cm extent.
- [ ] Change `SetBoxExtent(FVector(140.0f, 140.0f, 160.0f))` to `SetBoxExtent(FVector(250.0f, 250.0f, 160.0f))`.
- [ ] Rebuild and rerun the test to confirm it passes.

### Task 2: Hide duplicate close button in interaction menu

**Files:**
- Modify: `Source/Dispatch/Public/Widget/DialogueScreenWidget.h`
- Modify: `Source/Dispatch/Private/Widget/DialogueScreenWidget.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCDialogueInteractionTests.cpp`

**Interfaces:**
- Produces: `static bool ShouldDisplayCloseButton(const UDialogueSubsystem* DialogueSubsystem)`
- Consumes: `CanPlayerCloseCurrentSession()` and `IsDialogueActive()`

- [ ] Extend the existing player-close policy test: Story returns false, the player interaction menu returns false, and an active player-interaction dialogue tree returns true.
- [ ] Run `Dispatch.Dialogue.Interaction.PlayerClosePolicy` and confirm compilation fails because `ShouldDisplayCloseButton` is missing.
- [ ] Implement the static policy as `DialogueSubsystem && DialogueSubsystem->CanPlayerCloseCurrentSession() && DialogueSubsystem->IsDialogueActive()`.
- [ ] Make `RefreshCloseButtonVisibility()` use the policy.
- [ ] Build `DispatchEditor` and run `Dispatch.Dialogue.Interaction.PlayerClosePolicy`, `Dispatch.Dialogue.UI.PresentationContract`, and `Dispatch.NPC.Character.DialogueInteractPrompt`.
