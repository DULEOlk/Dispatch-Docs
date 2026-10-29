# Quest Summary Data Flow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a plain-text `QuestSummary` authored in `DT_Quests` and preserve it through the notice-board and notebook UI data flow.

**Architecture:** `FQuestContentRow` owns the authored value. `UQuestContentSubsystem` resolves it into `FResolvedQuestContent`, `ANoticeBoard` copies it into `FNoticeQuest`, and `MakeQuestInfoFromNoticeQuest` copies it into `FQuestInfo`. `UQuestListItemWidget` binds only that field to `Text_Summary`; rich `QuestDescription` remains exclusive to detail views.

**Tech Stack:** Unreal Engine 5.6, C++ USTRUCT reflection, UGameInstanceSubsystem, UMG, Unreal Automation Tests.

## Global Constraints

- Do not modify or save any `.uasset` during the code implementation.
- Preserve all pre-existing dirty workspace changes.
- An empty `QuestSummary` must remain empty and must not fall back to `QuestDescription`.
- `QuestDescription` remains rich text and may contain generated `<clue>` tags.

---

### Task 1: Define and resolve the summary data contract

**Files:**
- Modify: `Source/Dispatch/Public/Quest/ReadableContentTypes.h`
- Modify: `Source/Dispatch/Public/Quest/QuestTypes.h`
- Modify: `Source/Dispatch/Public/Items/NoticeBoard.h`
- Modify: `Source/Dispatch/Private/Tests/QuestContentSubsystemTests.cpp`
- Modify: `Source/Dispatch/Private/Quest/QuestContentSubsystem.cpp`

**Interfaces:**
- Produces: `FText QuestSummary` on `FQuestContentRow`, `FResolvedQuestContent`, `FNoticeQuest`, and `FQuestInfo`.
- Produces: `UQuestContentSubsystem::ResolveQuestContent` copies `FQuestContentRow::QuestSummary` to `FResolvedQuestContent::QuestSummary`.

- [ ] **Step 1: Write the failing reflected-contract and resolver assertions**

Add `Quest.QuestSummary = FText::FromString(TEXT("Trace the missing merchant."));` in `MakeQuestTable`, then assert:

```cpp
TestNotNull(
    TEXT("Resolved quest content exposes QuestSummary"),
    FindFProperty<FTextProperty>(FResolvedQuestContent::StaticStruct(), TEXT("QuestSummary")));
TestEqual(
    TEXT("Resolved content retains summary"),
    Resolved.QuestSummary.ToString(),
    FString(TEXT("Trace the missing merchant.")));
```

- [ ] **Step 2: Build to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: compilation fails because `QuestSummary` does not exist.

- [ ] **Step 3: Add the four reflected fields and resolver copy**

Add this property in the relevant category of each struct:

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest|Notebook", meta = (MultiLine = "true"))
FText QuestSummary;
```

For read-only resolved content, use `BlueprintReadOnly`. Copy it in `ResolveQuestContent`:

```cpp
OutContent.QuestSummary = Quest->QuestSummary;
```

- [ ] **Step 4: Build and run the content test**

Run the editor build, then:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Quest.Content;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: build succeeds and content tests pass.

### Task 2: Preserve summary through notice-board conversion

**Files:**
- Modify: `Source/Dispatch/Private/Tests/NoticeBoardTests.cpp`
- Modify: `Source/Dispatch/Private/Items/NoticeBoard.cpp`

**Interfaces:**
- Consumes: `FResolvedQuestContent::QuestSummary`.
- Produces: `FNoticeQuest::QuestSummary` and `FQuestInfo::QuestSummary`.

- [ ] **Step 1: Write failing propagation assertions**

Set the table fixture summary and assert both conversion boundaries:

```cpp
Quest.QuestSummary = FText::FromString(TEXT("Trace the missing merchant."));
TestEqual(
    TEXT("Notice keeps summary"),
    NoticeQuest.QuestSummary.ToString(),
    FString(TEXT("Trace the missing merchant.")));
TestEqual(
    TEXT("Quest info keeps summary"),
    QuestInfo.QuestSummary.ToString(),
    FString(TEXT("Trace the missing merchant.")));
```

- [ ] **Step 2: Run the notice-board test to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Items.NoticeBoard;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: assertions fail because the two copy assignments are absent.

- [ ] **Step 3: Add the two assignments**

In `SetQuestSlotByID`:

```cpp
Quest.QuestSummary = Content.QuestSummary;
```

In `MakeQuestInfoFromNoticeQuest`:

```cpp
QuestInfo.QuestSummary = Quest.QuestSummary;
```

- [ ] **Step 4: Rebuild and rerun the notice-board test**

Expected: notice-board tests pass.

### Task 3: Bind the compact list to QuestSummary only

**Files:**
- Modify: `Source/Dispatch/Private/Tests/QuestNotebookWidgetTests.cpp`
- Modify: `Source/Dispatch/Private/Widget/QuestListItemWidget.cpp`

**Interfaces:**
- Consumes: `FQuestInfo::QuestSummary`.
- Produces: `Text_Summary` contains plain summary text without rich-text markup.

- [ ] **Step 1: Write the failing widget assertion**

Bind a real `UTextBlock` to the reflected optional property, configure distinct summary and rich description values, and assert the summary widget:

```cpp
UTextBlock* SummaryText = NewObject<UTextBlock>(Row);
FObjectPropertyBase* SummaryProperty =
    FindFProperty<FObjectPropertyBase>(Row->GetClass(), TEXT("Text_Summary"));
SummaryProperty->SetObjectPropertyValue_InContainer(Row, SummaryText);

Quest.QuestSummary = FText::FromString(TEXT("Trace the missing merchant."));
Quest.QuestDescription = FText::FromString(
    TEXT("Read <clue id=\"Clue.Day01.MerchantMissing\">the clue</>."));
Row->ConfigureQuestItem(Quest);

TestEqual(
    TEXT("Compact row uses plain quest summary"),
    SummaryText->GetText().ToString(),
    FString(TEXT("Trace the missing merchant.")));
TestFalse(
    TEXT("Compact row does not expose clue markup"),
    SummaryText->GetText().ToString().Contains(TEXT("<clue")));
```

- [ ] **Step 2: Run the notebook widget test to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Widget.QuestNotebook;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: the row shows `QuestDescription` instead of `QuestSummary`.

- [ ] **Step 3: Change the binding**

Replace the list binding with:

```cpp
Text_Summary->SetText(QuestData.QuestSummary);
```

- [ ] **Step 4: Rebuild and run regression suites**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Quest;Automation RunTests Dispatch.Items.NoticeBoard;Automation RunTests Dispatch.Widget.QuestNotebook;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: all targeted suites pass.

### Task 4: Editor handoff

**Files:**
- No automated `.uasset` edits.

**Interfaces:**
- Consumes: reflected `FQuestContentRow::QuestSummary`.
- Produces: an editable `Quest Summary` field in every `DT_Quests` row after the editor reloads the rebuilt module.

- [ ] **Step 1: Verify the reflected field exists**

Confirm the data-contract automation test finds `QuestSummary` on `FQuestInfo`.

- [ ] **Step 2: Give the editor authoring instruction**

After closing/reopening the editor or reloading the C++ module, open `DT_Quests`, select `Quest.Day01`, and fill:

```text
调查失踪商人的最后行踪。
```

Save `DT_Quests`. No StateTree changes are required because StateTree still publishes only `QuestID`.
