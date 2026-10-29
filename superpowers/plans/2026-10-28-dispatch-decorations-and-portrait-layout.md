# Dispatch Decorations and Portrait Layout Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Separate the eight DispatchPages decoration textures from the four page widgets and preserve the user's authored NPC portrait layout in all three page widgets.

**Architecture:** Use the existing Unreal editor commandlet/asset APIs to move referenced textures and save only the affected widgets, with timestamped backups. Remove the 1.20 portrait maintenance behavior from the active authoring path and assert the authored slot geometry in focused editor tests.

**Tech Stack:** Unreal Engine 5.6, C++, editor commandlet, automation tests, PowerShell.

**Spec:** \`docs/superpowers/specs/2026-10-28-dispatch-decorations-and-portrait-layout.md\`

## Global Constraints

- Preserve existing source, Blueprint, map, and art changes.
- Do not modify formal commission DataAssets, dispatch settlement, save data, or story content.
- Keep the four Widget assets in \`Content/BluePrints/Widget/Menu/Quest/DispatchPages/\`.
- Use the authored portrait geometry \`(31.334627, 76.684608, 215.807251, 248.432007)\` for all three page widgets.

## Review Focus

- Every moved texture reference resolves after an editor restart.
- The three page widgets retain their authored portrait slot geometry after \`NativeConstruct\`.
- The old 1.20 portrait scaling helper cannot be invoked by the active preview path.
- Existing page mode and NPC portrait binding tests remain green.

### Task 1: Add asset organization and authored portrait audit

**Files:**
- Modify: \`Source/DispatchEditor/Private/CommissionCardEffectsCommandlet.cpp\`
- Modify: \`Source/DispatchEditor/Private/DecoratedDispatchReview.cpp\`
- Test: existing dispatch asset/page tests

- [ ] Add an explicit editor command for moving the eight named textures into \`Content/UI/Commission/Dispatch/Decorations/{Paper,Dividers,InkOrOrnaments}\` while backing up the eight textures and four widgets.
- [ ] Add an audit assertion for the three \`Image_NPCPortrait\` canvas slots against the authored geometry.
- [ ] Build the editor target and run the command in a fresh editor process.

### Task 2: Verify moved references and portrait behavior

**Files:**
- Test: existing \`CommissionDispatchPageTests\`, \`InProgressPresentation\`, and focused dispatch RHI tests

- [ ] Run an independent asset registry/reference audit.
- [ ] Run the focused real-RHI dispatch and portrait tests.
- [ ] Inspect the resulting page screenshot and record the backup/log paths in the current handoff.

