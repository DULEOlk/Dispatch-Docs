# Story Workspace Editor Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a dedicated `DispatchEditor` module and make `UStoryCampaignDefinition` open a unified Story Workspace whose first functional tab is the authoritative content library.

**Architecture:** Editor-only asset creation, scanning, catalog mutation, validation display, and Slate UI live in `DispatchEditor`. A testable service layer performs all content operations; the Slate widgets call that layer and never manipulate typed catalog arrays independently. Opening a Campaign uses a custom asset action to launch one four-tab `FAssetEditorToolkit`.

**Tech Stack:** Unreal Engine 5.6 editor modules, `FAssetEditorToolkit`, Slate, Details View, AssetTools, Asset Registry, Content Browser, ToolMenus, Unreal Automation Tests.

**Spec:** `C:\Users\DULEOLK\.codex\visualizations\2026\08\06\019fd933-cca8-7a52-aeaa-ae3639ae03f7\2026-08-26-story-workspace-content-authoring-design.md`

## Global Constraints

- This plan starts only after the Story Content Foundation plan passes.
- `DispatchEditor` is type `Editor` and is never a runtime or Shipping dependency.
- Catalog arrays are the only authoritative formal-content list; Asset Registry scans only discover candidates.
- Removing content from the Catalog never deletes the `.uasset`.
- `Items` is shown as unavailable until a formal item Definition exists.
- The Narrative tab does not invent a second graph system; it shows the existing Campaign/Episode summary until the later graph plan.
- The Flow tab opens the Campaign `RootStateTree`; it does not copy StateTree data into the editor.
- Every service task is automation-tested before Slate is connected.

---

## File Structure

### Module and registration

- `Dispatch.uproject` — register the `DispatchEditor` module.
- `Source/DispatchEditor.Target.cs` — include `DispatchEditor` for editor builds.
- `Source/DispatchEditor/DispatchEditor.Build.cs` — editor-only dependencies.
- `Source/DispatchEditor/Public/DispatchEditorModule.h` / `Private/DispatchEditorModule.cpp` — module lifecycle and asset-action registration.
- `Source/DispatchEditor/Private/AssetTypeActions/AssetTypeActions_StoryCampaign.h/.cpp` — open Story Workspace for Campaign assets.

### Testable editor services

- `Source/DispatchEditor/Public/StoryWorkspace/StoryContentEditorTypes.h` — categories, list items, and validation messages.
- `Source/DispatchEditor/Public/StoryWorkspace/StoryContentCatalogEditorService.h` / `Private/...cpp` — enumerate, filter, add, remove, create, scan, and validate.
- `Source/DispatchEditor/Public/StoryWorkspace/StoryWorkspaceEditorSettings.h` — content scan root and later preview defaults.
- `Source/DispatchEditor/Private/Tests/StoryContentCatalogEditorServiceTests.cpp` — service tests.

### Toolkit and Slate

- `Source/DispatchEditor/Public/StoryWorkspace/StoryWorkspaceEditor.h` / `Private/...cpp` — asset toolkit and tab registration.
- `Source/DispatchEditor/Private/StoryWorkspace/SStoryContentLibrary.h/.cpp` — functional content library.
- `Source/DispatchEditor/Private/StoryWorkspace/SStoryWorkspaceSummary.h/.cpp` — concrete Campaign/Narrative and Flow summaries.
- `Source/DispatchEditor/Private/StoryWorkspace/SStoryValidationPreview.h/.cpp` — validation list and preview area shell used by the next plan.
- `Source/DispatchEditor/Private/Tests/StoryWorkspaceEditorTests.cpp` — open behavior and tab smoke tests.

---

### Task 1: Register the editor module and Campaign asset action

**Files:**
- Modify: `Dispatch.uproject`
- Modify: `Source/DispatchEditor.Target.cs`
- Create: `Source/DispatchEditor/DispatchEditor.Build.cs`
- Create: `Source/DispatchEditor/Public/DispatchEditorModule.h`
- Create: `Source/DispatchEditor/Private/DispatchEditorModule.cpp`
- Create: `Source/DispatchEditor/Private/AssetTypeActions/AssetTypeActions_StoryCampaign.h`
- Create: `Source/DispatchEditor/Private/AssetTypeActions/AssetTypeActions_StoryCampaign.cpp`

**Interfaces:**
- Produces: module `DispatchEditor`.
- Produces: `FAssetTypeActions_StoryCampaign::OpenAssetEditor` calling `FStoryWorkspaceEditor::OpenEditor(...)`.

- [ ] **Step 1: Register a failing module-load automation test**

Create `Source/DispatchEditor/Private/Tests/StoryWorkspaceModuleTests.cpp` with:

```cpp
IMPLEMENT_SIMPLE_AUTOMATION_TEST(
    FStoryWorkspaceModuleLoadsTest,
    "Dispatch.Editor.StoryWorkspace.ModuleLoads",
    EAutomationTestFlags::EditorContext | EAutomationTestFlags::EngineFilter)

bool FStoryWorkspaceModuleLoadsTest::RunTest(const FString& Parameters)
{
    TestTrue(TEXT("DispatchEditor is loaded"), FModuleManager::Get().IsModuleLoaded(TEXT("DispatchEditor")));
    return true;
}
```

- [ ] **Step 2: Build and verify the module is missing**

Run the editor build. Expected: the project cannot compile the new test because the module directory and rules do not exist.

- [ ] **Step 3: Add module descriptors and dependencies**

Append this project module descriptor:

```json
{
  "Name": "DispatchEditor",
  "Type": "Editor",
  "LoadingPhase": "Default"
}
```

Add `DispatchEditor` to `ExtraModuleNames` in `DispatchEditor.Target.cs`. Set editor-module dependencies to:

```csharp
PublicDependencyModuleNames.AddRange(new[] { "Core", "CoreUObject", "Engine", "Dispatch" });
PrivateDependencyModuleNames.AddRange(new[] {
    "ApplicationCore", "AssetRegistry", "AssetTools", "ContentBrowser",
    "EditorFramework", "EditorSubsystem", "InputCore", "PropertyEditor",
    "Slate", "SlateCore", "ToolMenus", "UnrealEd"
});
```

Do not move current runtime dependencies in this task.

- [ ] **Step 4: Implement module lifecycle and safe asset-action ownership**

`StartupModule` loads `AssetTools`, creates one `TSharedPtr<FAssetTypeActions_StoryCampaign>`, and registers it. `ShutdownModule` first checks `AssetTools` is loaded, unregisters the same instance, and resets the pointer. Use `IMPLEMENT_MODULE(FDispatchEditorModule, DispatchEditor)`.

The asset action reports `Story Campaign`, uses `UStoryCampaignDefinition::StaticClass()`, and forwards every valid selected Campaign to `FStoryWorkspaceEditor::OpenEditor`.

- [ ] **Step 5: Build and run module-load test**

Run `Dispatch.Editor.StoryWorkspace.ModuleLoads`. Expected: pass.

- [ ] **Step 6: Commit**

```powershell
git add Dispatch.uproject Source/DispatchEditor.Target.cs Source/DispatchEditor
git commit -m "feat: add Dispatch editor module"
```

### Task 2: Implement the content-catalog editor service

**Files:**
- Create: `Source/DispatchEditor/Public/StoryWorkspace/StoryContentEditorTypes.h`
- Create: `Source/DispatchEditor/Public/StoryWorkspace/StoryContentCatalogEditorService.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/StoryContentCatalogEditorService.cpp`
- Create: `Source/DispatchEditor/Public/StoryWorkspace/StoryWorkspaceEditorSettings.h`
- Create: `Source/DispatchEditor/Private/Tests/StoryContentCatalogEditorServiceTests.cpp`

**Interfaces:**
- Consumes: `UStoryContentCatalog` and all supported Definition classes.
- Produces: `EStoryContentCategory`, `FStoryContentListItem`, `FStoryContentValidationMessage`.
- Produces: service methods `GetItems`, `AddExisting`, `RemoveReference`, `ScanUnregistered`, `CreateAndRegister`, and `ValidateWorkspace`.

- [ ] **Step 1: Write failing pure-service tests**

Test these operations on transient packages and assets:

```cpp
TestTrue(TEXT("Add accepts matching clue"), Service.AddExisting(Catalog, EStoryContentCategory::Clue, Clue, Error));
TestFalse(TEXT("Add rejects duplicate path"), Service.AddExisting(Catalog, EStoryContentCategory::Clue, Clue, Error));
TestFalse(TEXT("Add rejects wrong category"), Service.AddExisting(Catalog, EStoryContentCategory::Dialogue, Clue, Error));
TestTrue(TEXT("Remove only removes reference"), Service.RemoveReference(Catalog, Clue));
TestFalse(TEXT("Remove does not destroy asset"), Clue->IsUnreachable());
TestEqual(TEXT("Search filters case-insensitively"), Service.GetItems(Catalog, EStoryContentCategory::Clue, TEXT("catalog a")).Num(), 1);
```

Use `/Game/Developers/StoryWorkspaceTests` assets for Asset Registry scan tests and delete only those exact test packages in test cleanup.

- [ ] **Step 2: Run the focused build and confirm missing-service failure**

Expected: compilation fails on the new editor types and service.

- [ ] **Step 3: Implement category descriptors and one mutation path**

Define:

```cpp
enum class EStoryContentCategory : uint8
{
    Letter, Dialogue, NPC, Clue, Inference, Commission, Item, Day
};

struct FStoryContentListItem
{
    FSoftObjectPath AssetPath;
    FName ContentID;
    FText DisplayName;
    EStoryContentCategory Category;
    bool bLoaded = false;
};
```

Create a single internal category descriptor that maps category to expected `UClass`, display text, and typed Catalog array operations. Every public add/remove/scan/create operation must use this descriptor. The Item descriptor has no class and returns the explicit error `物品尚未迁移为正式 Definition`.

- [ ] **Step 4: Implement create, add, remove, enumerate, and scan**

- `AddExisting` verifies class/category, rejects duplicate soft paths, calls `Catalog->Modify()`, adds through the descriptor, and calls `MarkPackageDirty()`.
- `RemoveReference` calls `Modify`, removes the exact soft path, marks the Catalog dirty, and never calls asset deletion.
- `CreateAndRegister` creates the category class using `UDataAssetFactory`, ensures the requested package path is under `/Game`, then immediately calls `AddExisting`.
- `ScanUnregistered` queries `IAssetRegistry` under `UStoryWorkspaceEditorSettings::ContentScanRoot` recursively, filters by descriptor class, and excludes paths already in the Catalog.
- `GetItems` returns deterministic category/ID/path ordering and applies a case-insensitive match against ID, display name, and asset path.

- [ ] **Step 5: Implement validation messages**

`ValidateWorkspace(UStoryCampaignDefinition* Campaign)` validates the assigned Catalog through runtime `ValidateDefinition`, then returns `FStoryContentValidationMessage` entries with severity, message, and best available `FSoftObjectPath`. It also adds editor-only warnings when the Campaign has no Catalog, has no `RootStateTree`, or when the reserved Item category is selected; these warnings do not make runtime Campaign validation fail.

- [ ] **Step 6: Run service tests**

Run `Dispatch.Editor.StoryWorkspace.ContentService`. Expected: all mutation, scanning, filtering, and validation cases pass.

- [ ] **Step 7: Commit**

```powershell
git add Source/DispatchEditor/Public/StoryWorkspace Source/DispatchEditor/Private/StoryWorkspace/StoryContentCatalogEditorService.cpp Source/DispatchEditor/Private/Tests/StoryContentCatalogEditorServiceTests.cpp
git commit -m "feat: add story content catalog editor service"
```

### Task 3: Build the functional Content Library tab

**Files:**
- Create: `Source/DispatchEditor/Private/StoryWorkspace/SStoryContentLibrary.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/SStoryContentLibrary.cpp`
- Modify: `Source/DispatchEditor/DispatchEditor.Build.cs`

**Interfaces:**
- Consumes: `FStoryContentCatalogEditorService`.
- Produces: a Slate widget whose selection exposes `UObject* GetSelectedAsset() const` and `FOnStoryContentSelectionChanged`.

- [ ] **Step 1: Add a Slate construction smoke test**

Create a transient Campaign and Catalog, construct `SStoryContentLibrary`, and assert the widget and its initial `Letter` category exist without loading a map. Expected initial item count: zero.

- [ ] **Step 2: Run the smoke test and verify missing-widget failure**

Expected: compilation fails because `SStoryContentLibrary` does not exist.

- [ ] **Step 3: Implement the three-column layout**

Construct exact regions:

- left `SListView` categories: 信件、对话、NPC、线索、推论、委托、物品、日期与撰写;
- center toolbar with `SSearchBox` and buttons 新建、添加已有、扫描未登记、移出目录、定位;
- center `SListView<TSharedPtr<FStoryContentListItem>>` with display name, ID, path, and validation marker;
- right `IDetailsView` created from `PropertyEditor`, hidden name area, selected asset as object;
- bottom `SListView` for validation/reference messages.

Selecting Item shows the unavailable message and disables New/Add/Scan. Selecting another category refreshes through `Service.GetItems`.

- [ ] **Step 4: Wire commands to the service**

- New opens a save-asset dialog, calls `CreateAndRegister`, then selects the created asset.
- Add Existing opens a Content Browser asset picker restricted to the category class; confirmation calls `AddExisting`.
- Scan opens a confirmation window listing only `ScanUnregistered` results; the user checks rows and clicks 批量登记.
- Remove prompts `只从目录移除引用，不删除资产。继续？` and calls `RemoveReference`.
- Locate calls `GEditor->SyncBrowserToObjects` with the loaded selected object.
- Search/category changes rebuild list items; asset edits trigger validation and refresh via `FCoreUObjectDelegates::OnObjectPropertyChanged` filtered to the selected object and Catalog.

- [ ] **Step 5: Run Slate smoke and service tests**

Run `Dispatch.Editor.StoryWorkspace.ContentLibrary` and `ContentService`. Expected: pass.

- [ ] **Step 6: Commit**

```powershell
git add Source/DispatchEditor/Private/StoryWorkspace/SStoryContentLibrary.h Source/DispatchEditor/Private/StoryWorkspace/SStoryContentLibrary.cpp Source/DispatchEditor/DispatchEditor.Build.cs Source/DispatchEditor/Private/Tests
git commit -m "feat: add Story Workspace content library"
```

### Task 4: Add the four-tab Story Workspace toolkit

**Files:**
- Create: `Source/DispatchEditor/Public/StoryWorkspace/StoryWorkspaceEditor.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/StoryWorkspaceEditor.cpp`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/SStoryWorkspaceSummary.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/SStoryWorkspaceSummary.cpp`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/SStoryValidationPreview.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/SStoryValidationPreview.cpp`
- Create: `Source/DispatchEditor/Private/Tests/StoryWorkspaceEditorTests.cpp`

**Interfaces:**
- Consumes: Campaign, Catalog, content library.
- Produces: `FStoryWorkspaceEditor::OpenEditor(UStoryCampaignDefinition*, TSharedPtr<IToolkitHost>)`.
- Produces tab IDs: `StoryWorkspace.Content`, `StoryWorkspace.Narrative`, `StoryWorkspace.Flow`, `StoryWorkspace.ValidationPreview`.

- [ ] **Step 1: Write failing toolkit tab tests**

Construct a toolkit for a transient Campaign and assert all four tab spawners register. Verify the toolkit refuses a null Campaign and that the Content tab receives the exact `Campaign->ContentCatalog` pointer.

- [ ] **Step 2: Run the focused test and confirm missing-toolkit failure**

Expected: compilation fails because the toolkit does not exist.

- [ ] **Step 3: Implement toolkit lifetime and tabs**

Derive from `FAssetEditorToolkit` and `FGCObject`. `AddReferencedObjects` retains the edited Campaign. `InitAssetEditor` uses app identifier `DispatchStoryWorkspaceApp`, a standalone/hosted mode passed by the asset action, and a layout with Content taking 65% width plus a right stack containing Narrative, Flow, and Validation & Preview.

Tab behavior:

- Content: functional `SStoryContentLibrary`.
- Narrative: lists Campaign ID, Starting Episode, Episodes, and each episode day; text states that the later graph editor will visualize these existing definitions, not create a second runtime graph.
- Flow: displays `RootStateTree` path, warns if empty, and an `打开 StateTree` button that calls `GEditor->GetEditorSubsystem<UAssetEditorSubsystem>()->OpenEditorForAsset`.
- Validation & Preview: displays current catalog/campaign validation messages and a disabled preview section reading `选择支持预览的内容后可启动`; the next plan enables it.

- [ ] **Step 4: Implement cleanup**

Unregister all tab spawners in the toolkit destructor, remove property-change delegates, and release Details View references. Opening the same Campaign again should focus the existing asset editor through Unreal's asset editor subsystem instead of creating a duplicate toolkit.

- [ ] **Step 5: Run toolkit and module tests**

Run `Dispatch.Editor.StoryWorkspace`. Expected: module, service, library, and toolkit tests all pass.

- [ ] **Step 6: Commit**

```powershell
git add Source/DispatchEditor/Public/StoryWorkspace/StoryWorkspaceEditor.h Source/DispatchEditor/Private/StoryWorkspace Source/DispatchEditor/Private/Tests/StoryWorkspaceEditorTests.cpp
git commit -m "feat: add unified Story Workspace editor"
```

### Task 5: Create the Demo workspace assets through an editor command

**Files:**
- Create: `Source/DispatchEditor/Private/StoryWorkspace/StoryWorkspaceBootstrap.h`
- Create: `Source/DispatchEditor/Private/StoryWorkspace/StoryWorkspaceBootstrap.cpp`
- Modify: `Source/DispatchEditor/Private/DispatchEditorModule.cpp`
- Test/Create: `Content/BluePrints/Story/Definition/Narrative/DA_StoryContentCatalog_Demo.uasset`
- Test/Create: `Content/BluePrints/Story/Definition/Narrative/DA_StoryCampaign_Demo.uasset`

**Interfaces:**
- Produces console command: `StoryWorkspace.CreateDemoAssets`.
- Produces exact package paths for the Demo catalog and Campaign.

- [ ] **Step 1: Write a bootstrap idempotency test**

Invoke `FStoryWorkspaceBootstrap::CreateDemoAssets`, assert both assets exist at the exact paths, invoke it again, and assert it returns the same objects without duplicating or overwriting existing fields.

- [ ] **Step 2: Run the test and confirm missing-bootstrap failure**

Expected: compilation fails because the bootstrap does not exist.

- [ ] **Step 3: Implement non-destructive asset creation**

Use `AssetTools.CreateAsset` with `UDataAssetFactory` only when the Asset Registry path does not exist. Assign the new Catalog to a newly created Campaign. Do not invent episodes, content, or a Root StateTree. If the Campaign already exists, only fill `ContentCatalog` when it is null; never clear user data. Save only packages created or modified by this function.

Register the command in `StartupModule` and unregister it in `ShutdownModule`.

- [ ] **Step 4: Generate the assets**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=StoryWorkspace.CreateDemoAssets;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: both `.uasset` files exist; the Campaign opens in Story Workspace and shows expected authoring warnings for missing episode/flow data rather than crashing.

- [ ] **Step 5: Run editor tests and full regression**

Run the editor build, `Dispatch.Editor.StoryWorkspace`, then the complete `Dispatch` suite. Expected: all pass.

- [ ] **Step 6: Commit**

```powershell
git add Source/DispatchEditor/Private/StoryWorkspace/StoryWorkspaceBootstrap.h Source/DispatchEditor/Private/StoryWorkspace/StoryWorkspaceBootstrap.cpp Source/DispatchEditor/Private/DispatchEditorModule.cpp Content/BluePrints/Story/Definition/Narrative/DA_StoryContentCatalog_Demo.uasset Content/BluePrints/Story/Definition/Narrative/DA_StoryCampaign_Demo.uasset
git commit -m "feat: add demo Story Workspace assets"
```
