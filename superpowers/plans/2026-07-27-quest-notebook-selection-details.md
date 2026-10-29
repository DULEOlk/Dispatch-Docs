# Quest Notebook Selection and Details Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add tested C++ parents for the quest-list row and player notebook, including exclusive red selection state, fixed-layout details, and three quest-specific images.

**Architecture:** Extend `FQuestInfo` with notebook presentation data. `UQuestListItemWidget` owns one row's bindings and selected visual, while `UPlayerNotebookWidget` creates rows, owns exclusive selection, and populates the shared detail panel.

**Tech Stack:** Unreal Engine 5.6, C++20/UHT, UMG, Slate/SlateCore, Unreal Editor automation tests.

## Global Constraints

- `WBP_QuestListItem` and `WBP_PlayerNotebook` each receive their own direct `UUserWidget` C++ parent.
- No quest is selected automatically when the list is built or rebuilt.
- The “委托详情” heading and divider remain visible while `Panel_QuestDetailContent` remains collapsed until selection.
- Exactly three quest-authored images are supported: list icon, detail illustration, and map image.
- Existing unsaved `.uasset` files are not rewritten by command-line automation.
- Existing unrelated worktree changes are preserved.

---

### Task 1: Extend the notebook quest data contract

**Files:**
- Modify: `Source/Dispatch/Public/Quest/QuestTypes.h`
- Create: `Source/Dispatch/Private/Tests/QuestNotebookWidgetTests.cpp`

**Interfaces:**
- Produces: `FQuestInfo::QuestLocation`, `QuestState`, `QuestListIcon`, `QuestDetailImage`, and `QuestMapImage`.
- Consumes: existing `EQuestRuntimeState` and `FQuestInfo`.

- [ ] **Step 1: Write the failing reflection test**

Add an automation test named `Dispatch.Widget.QuestNotebook.DataContract`:

```cpp
IMPLEMENT_SIMPLE_AUTOMATION_TEST(
	FQuestNotebookDataContractTest,
	"Dispatch.Widget.QuestNotebook.DataContract",
	EAutomationTestFlags::EditorContext | EAutomationTestFlags::EngineFilter)

bool FQuestNotebookDataContractTest::RunTest(const FString& Parameters)
{
	const UScriptStruct* QuestInfo = FQuestInfo::StaticStruct();
	TestNotNull(TEXT("QuestLocation exists"), FindFProperty<FTextProperty>(QuestInfo, TEXT("QuestLocation")));
	TestNotNull(TEXT("QuestState exists"), FindFProperty<FEnumProperty>(QuestInfo, TEXT("QuestState")));
	TestNotNull(TEXT("QuestListIcon exists"), FindFProperty<FSoftObjectProperty>(QuestInfo, TEXT("QuestListIcon")));
	TestNotNull(TEXT("QuestDetailImage exists"), FindFProperty<FSoftObjectProperty>(QuestInfo, TEXT("QuestDetailImage")));
	TestNotNull(TEXT("QuestMapImage exists"), FindFProperty<FSoftObjectProperty>(QuestInfo, TEXT("QuestMapImage")));
	return true;
}
```

- [ ] **Step 2: Build and run the test to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Then run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Widget.QuestNotebook.DataContract;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: the build succeeds and the automation test fails because the five reflected fields do not exist.

- [ ] **Step 3: Add the minimal fields**

Add to `FQuestInfo`:

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest|Notebook")
FText QuestLocation;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest|Notebook")
EQuestRuntimeState QuestState = EQuestRuntimeState::None;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest|Notebook")
TSoftObjectPtr<UTexture2D> QuestListIcon;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest|Notebook")
TSoftObjectPtr<UTexture2D> QuestDetailImage;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest|Notebook")
TSoftObjectPtr<UTexture2D> QuestMapImage;
```

Forward-declare `UTexture2D`.

- [ ] **Step 4: Build to verify GREEN**

Run the same `Build.bat` command. Expected: exit code `0`.

---

### Task 2: Implement the quest-list row C++ parent

**Files:**
- Create: `Source/Dispatch/Public/Widget/QuestListItemWidget.h`
- Create: `Source/Dispatch/Private/Widget/QuestListItemWidget.cpp`
- Modify: `Source/Dispatch/Private/Tests/QuestNotebookWidgetTests.cpp`

**Interfaces:**
- Consumes: `FQuestInfo`.
- Produces:
  - `void ConfigureQuestItem(const FQuestInfo& InQuestData)`
  - `void SetQuestSelected(bool bSelected)`
  - `bool IsQuestSelected() const`
  - `const FQuestInfo& GetQuestData() const`
  - `static FText GetQuestStateDisplayText(EQuestRuntimeState State)`
  - `FOnQuestItemClicked OnQuestItemClicked`

- [ ] **Step 1: Write failing row behavior tests**

Create a real row with `NewObject<UQuestListItemWidget>()`, configure literal quest data, and assert:

```cpp
TestEqual(TEXT("Row stores quest ID"), Row->GetQuestData().QuestID, FName(TEXT("Quest.Deer")));
TestFalse(TEXT("Row begins unselected"), Row->IsQuestSelected());
Row->SetQuestSelected(true);
TestTrue(TEXT("Row can be selected"), Row->IsQuestSelected());
Row->SetQuestSelected(false);
TestFalse(TEXT("Row can be deselected"), Row->IsQuestSelected());
TestEqual(
	TEXT("Accepted displays in-progress text"),
	UQuestListItemWidget::GetQuestStateDisplayText(EQuestRuntimeState::Accepted).ToString(),
	FString(TEXT("进行中")));
```

Also reflect the expected optional widget bindings and callable functions.

- [ ] **Step 2: Build to verify RED**

Run the editor build. Expected: compilation fails because `UQuestListItemWidget` does not exist.

- [ ] **Step 3: Add the minimal row parent**

Declare `UQuestListItemWidget : public UUserWidget`, its dynamic multicast delegate, optional `BindWidget` properties, stored data, and selected bool. Override `NativeOnInitialized`, bind `Button_Quest`, and initialize the marker to collapsed.

`ConfigureQuestItem` sets:

```cpp
Text_Title->SetText(QuestData.QuestTitle);
Text_Summary->SetText(QuestData.QuestDescription);
Text_Location->SetText(QuestData.QuestLocation);
Text_Status->SetText(GetQuestStateDisplayText(QuestData.QuestState));
```

Use a private image helper that synchronously resolves the soft texture only when the widget is being populated. Empty/failed textures collapse their image.

- [ ] **Step 4: Build and run the focused tests**

Build the editor target, then run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Widget.QuestNotebook;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: the data-contract and row tests pass with zero errors.

---

### Task 3: Implement the notebook C++ parent and exclusive selection

**Files:**
- Create: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Create: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`
- Modify: `Source/Dispatch/Private/Tests/QuestNotebookWidgetTests.cpp`

**Interfaces:**
- Consumes: `UQuestListItemWidget`, `FQuestInfo`, and configured `QuestListItemClass`.
- Produces:
  - `void BuildQuestList(const TArray<FQuestInfo>& QuestArray)`
  - `bool SelectQuestItem(UQuestListItemWidget* QuestItem)`
  - `void ClearQuestSelection()`
  - `UQuestListItemWidget* GetSelectedQuestItem() const`
  - `bool HasQuestSelection() const`

- [ ] **Step 1: Write failing notebook selection tests**

Use real notebook and row objects. Register rows through the same internal path used by list construction and assert:

```cpp
TestFalse(TEXT("Notebook begins without selection"), Notebook->HasQuestSelection());
TestTrue(TEXT("First registered row can be selected"), Notebook->SelectQuestItem(First));
TestTrue(TEXT("First row selected"), First->IsQuestSelected());
TestTrue(TEXT("Second registered row can be selected"), Notebook->SelectQuestItem(Second));
TestFalse(TEXT("Selecting second row deselects first"), First->IsQuestSelected());
TestTrue(TEXT("Second row selected"), Second->IsQuestSelected());
TestTrue(TEXT("Repeated selection is accepted"), Notebook->SelectQuestItem(Second));
TestEqual(TEXT("Selected row remains second"), Notebook->GetSelectedQuestItem(), Second);
TestFalse(TEXT("Unregistered row is rejected"), Notebook->SelectQuestItem(Foreign));
Notebook->ClearQuestSelection();
TestFalse(TEXT("Clear removes selection"), Notebook->HasQuestSelection());
```

Expose `RegisterQuestItem` as a normal public method because it is also useful for Blueprint-authored/custom list construction; it must reject null and duplicate rows.

- [ ] **Step 2: Build to verify RED**

Run the editor build. Expected: compilation fails because `UPlayerNotebookWidget` does not exist.

- [ ] **Step 3: Add the minimal notebook parent**

Declare optional bindings for:

```cpp
UScrollBox* ScrollBox_QuestList;
UPanelWidget* Panel_QuestDetailContent;
UTextBlock* Text_Title;
URichTextBlock* Text_Description;
UTextBlock* Text_Meta;
UImage* QuestDetailImage;
UImage* QuestMapImage;
```

Store registered rows in a transient `TArray<TObjectPtr<UQuestListItemWidget>>` and selection in a transient `TObjectPtr<UQuestListItemWidget>`.

`BuildQuestList` must:

1. call `ClearQuestSelection`;
2. clear the scroll box and registered rows;
3. validate `ScrollBox_QuestList` and `QuestListItemClass`;
4. create/configure/register/add each row;
5. leave details collapsed.

`SelectQuestItem` must reject unregistered rows, avoid rebuilding details for the already selected row, deselect the old row, select the new row, populate details, and show `Panel_QuestDetailContent`.

- [ ] **Step 4: Build and run focused tests**

Run the editor build and `Dispatch.Widget.QuestNotebook` automation group. Expected: exit code `0` and no failed tests.

---

### Task 4: Verify integration and prepare the Blueprint handoff

**Files:**
- Verify: all files above
- No `.uasset` writes

**Interfaces:**
- Consumes: compiled C++ classes visible to Unreal Editor.
- Produces: a precise reparenting/control-name checklist for the user.

- [ ] **Step 1: Run source hygiene checks**

Run:

```powershell
git diff --check
git diff -- Source/Dispatch/Public/Quest/QuestTypes.h Source/Dispatch/Public/Widget/QuestListItemWidget.h Source/Dispatch/Private/Widget/QuestListItemWidget.cpp Source/Dispatch/Public/Widget/PlayerNotebookWidget.h Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp Source/Dispatch/Private/Tests/QuestNotebookWidgetTests.cpp
```

Expected: no whitespace errors and only scoped changes.

- [ ] **Step 2: Run a fresh full editor build**

Run the `DispatchEditor Win64 Development` build command. Expected: exit code `0`.

- [ ] **Step 3: Run the focused automation group**

Run the `Dispatch.Widget.QuestNotebook` group. Expected: all tests complete successfully with zero failures.

- [ ] **Step 4: Confirm Blueprint migration requirements**

Provide these exact handoff actions:

- reparent `WBP_QuestListItem` to `QuestListItemWidget`;
- rename the wax seal to `Image_Selected`;
- reparent `WBP_PlayerNotebook` to `PlayerNotebookWidget`;
- add `Panel_QuestDetailContent` around all changing detail controls;
- keep the heading/divider outside;
- use `QuestDetailImage` and `QuestMapImage`;
- set `QuestListItemClass` to `WBP_QuestListItem`;
- replace the old list/detail dispatcher graph with `BuildQuestList`.
