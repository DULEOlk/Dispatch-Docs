# NPC Mood State Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Keep the existing activity state for internal dispatch flow while adding a separate NPC mood state shown as the UI “状态” field.

**Architecture:** `ENPCCurrentState` remains the internal activity lifecycle (`空闲`, `执行委托中`, etc.). A new `ENPCMoodState` is authored on NPC definitions and copied into runtime NPC info. Dispatch and notebook presentation map the mood to the existing four-row “状态” slot; activity state is no longer displayed there.

**Tech Stack:** Unreal Engine C++, UPrimaryDataAsset NPC definitions, UMG presentation adapters, Unreal automation tests, editor commandlet asset authoring.

**Spec:** `docs/narrative/FOUNDATION_DATA_DESIGN.md`, `docs/narrative/SYSTEM_DESIGN.md`, current user-confirmed four-field layout.

## Global Constraints

- Preserve the existing activity-state enum and its serialized values.
- The display fields remain 健康、身体、状态、信任; 身体 is the commission suitability result.
- Formal NPC DataAsset writes are limited to the three existing NotebookPreview NPC assets and the new initial mood field; portraits, descriptions, commission associations, images, layout, and activity state remain unchanged.
- Save a backup before DataAsset authoring and run focused automation plus the existing notebook/dispatch UI regression.

## Review Focus

- Existing activity state still blocks or tracks commission flow after mood is added.
- Mood maps to the UI 状态 slot without falling back to 空闲.
- Missing/invalid mood data has a deterministic default.
- The three existing NPC assets retain their current activity state and other fields.

### Task 1: Mood contract test

**Files:**
- Modify: `Source/Dispatch/Private/Tests/NPCInfoComponentTests.cpp`

- [x] Add a failing automation test asserting `ENPCMoodState` exists with 平静、警惕、振奋、不安、低落、失控.
- [x] Run the focused automation test and observe the expected failure because the enum is not yet present.

### Task 2: Runtime and authored mood data

**Files:**
- Modify: `Source/Dispatch/Public/Component/NPCInfoComponent.h`
- Modify: `Source/Dispatch/Public/NPC/NPCDefinition.h`
- Modify: `Source/Dispatch/Private/NPC/NPCDefinition.cpp`
- Modify: `Source/Dispatch/Private/Tests/NPCDefinitionTests.cpp`

- [x] Add `ENPCMoodState` and initial/runtime mood fields with 平静 as the default.
- [x] Verify definition-to-runtime copying and preserve `ENPCCurrentState` behavior.
- [x] Build and run the focused NPC tests.

### Task 3: Presentation mapping

**Files:**
- Modify: `Source/Dispatch/Private/Widget/CommissionDispatchUIBase.cpp`
- Modify: `Source/Dispatch/Private/Widget/NotebookCommissionData.cpp`
- Modify or extend: focused presentation tests.

- [x] Map the display “状态” value from mood instead of activity state.
- [x] Keep the activity state available to runtime adapters but out of the four-row presentation.
- [x] Verify missing mood uses 平静 and existing health/trust/body mappings remain intact.

### Task 4: Author three existing NPC assets

**Files:**
- Modify: `/Game/Developers/NotebookPreview/NPC/DA_NPC_001_VeraKane`
- Modify: `/Game/Developers/NotebookPreview/NPC/DA_NPC_002_SeverinMoore`
- Modify: `/Game/Developers/NotebookPreview/NPC/DA_NPC_003_RolandGraeve`

- [x] Back up the three assets.
- [x] Set initial mood values and 256x256 portraits without changing activity state, descriptions, or commission associations.
- [x] Verify the saved assets and the recordbook/dispatch presentations.

### Task 5: Documentation and regression

- [x] Update `docs/narrative/CURRENT.md`, `docs/narrative/IMPLEMENTATION_MATRIX.md`, and the active notebook handoff.
- [x] Run the relevant Development build and real-RHI UI regression; record results and remaining manual review.
