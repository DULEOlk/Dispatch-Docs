# Player Notebook Modal Presentation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Hide and correctly restore the player HUD while the notebook is open, and place a medium full-screen blur with a 20% black tint behind the book.

**Architecture:** `UPlayerNotebookWidget` owns the modal lifecycle so every notebook opening path receives the same behavior. It captures the existing requested HUD state from `ADispatchPlayerController`, hides the HUD, creates one full-screen blur/tint layer below the Widget Blueprint content, and restores the captured state when the notebook closes.

**Tech Stack:** Unreal Engine 5.6, C++, UMG, Unreal Automation Tests.

## Global Constraints

- Preserve all pre-existing uncommitted project changes.
- Do not modify `WBP_PlayerNotebook.uasset` or `WBP_LetterUI.uasset`.
- Add modal tests in a new focused test file because `QuestNotebookWidgetTests.cpp` already contains unrelated user changes.
- Blur strength is `12.0`, overridden blur radius is `8`, and tint is black with alpha `0.20`.
- Restore the HUD state captured before the first activation; never force it visible unconditionally.

---

### Task 1: Notebook Modal Lifecycle and Presentation Layer

**Files:**
- Create: `Source/Dispatch/Private/Tests/PlayerNotebookModalTests.cpp`
- Modify: `Source/Dispatch/Public/Controller/DispatchPlayerController.h`
- Modify: `Source/Dispatch/Public/Widget/PlayerNotebookWidget.h`
- Modify: `Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp`

**Interfaces:**
- Consumes: `ADispatchPlayerController::SetPlayerHUDVisible(bool)`, the notebook Widget Blueprint's root `UCanvasPanel`, and `UUserWidget::NativeConstruct`/`NativeDestruct`.
- Produces: `bool ADispatchPlayerController::IsPlayerHUDRequestedVisible() const`, `void UPlayerNotebookWidget::ActivateModalPresentation(ADispatchPlayerController*)`, and `void UPlayerNotebookWidget::DeactivateModalPresentation()`.

- [ ] **Step 1: Verify the existing notebook test baseline**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Widget.QuestNotebook;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: build exit code `0`; all existing `Dispatch.Widget.QuestNotebook` tests pass.

- [ ] **Step 2: Write the failing modal behavior test**

Create `PlayerNotebookModalTests.cpp` with one real integration-style automation test. It must:

```cpp
Controller->SetPlayerHUDVisible(true);
Notebook->ActivateModalPresentation(Controller);
TestFalse(TEXT("Opening notebook hides requested HUD"), Controller->IsPlayerHUDRequestedVisible());

UBackgroundBlur* Blur =
	Cast<UBackgroundBlur>(Notebook->WidgetTree->FindWidget(TEXT("NotebookModalBackgroundBlur")));
TestNotNull(TEXT("Notebook creates a background blur"), Blur);
TestEqual(TEXT("Blur is medium strength"), Blur->GetBlurStrength(), 12.0f);
TestEqual(TEXT("Blur radius is eight"), Blur->GetBlurRadius(), 8);

UCanvasPanelSlot* BlurSlot = Cast<UCanvasPanelSlot>(Blur->Slot);
TestEqual(TEXT("Blur fills the root"), BlurSlot->GetAnchors(), FAnchors(0.0f, 0.0f, 1.0f, 1.0f));
TestEqual(TEXT("Blur has zero offsets"), BlurSlot->GetOffsets(), FMargin(0.0f));
TestTrue(TEXT("Blur is behind book content"), BlurSlot->GetZOrder() < 0);

UBorder* Tint = Cast<UBorder>(Blur->GetContent());
TestNotNull(TEXT("Blur owns a tint border"), Tint);
TestEqual(TEXT("Tint is twenty percent black"), Tint->GetBrushColor(), FLinearColor(0.0f, 0.0f, 0.0f, 0.20f));

Notebook->ActivateModalPresentation(Controller);
TestEqual(TEXT("Repeated activation creates no duplicate layer"), RootCanvas->GetChildrenCount(), 1);
Notebook->DeactivateModalPresentation();
TestTrue(TEXT("Closing restores initially visible HUD"), Controller->IsPlayerHUDRequestedVisible());

Controller->SetPlayerHUDVisible(false);
Notebook->ActivateModalPresentation(Controller);
Notebook->DeactivateModalPresentation();
TestFalse(TEXT("Closing preserves initially hidden HUD"), Controller->IsPlayerHUDRequestedVisible());
```

The test constructs a real editor test world, spawns a real `ADispatchPlayerController`, creates a real `UPlayerNotebookWidget`, and assigns a real `UCanvasPanel` to its `WidgetTree`.

- [ ] **Step 3: Build to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: compile failure because `ActivateModalPresentation`, `DeactivateModalPresentation`, or `IsPlayerHUDRequestedVisible` does not exist yet.

- [ ] **Step 4: Implement the minimal HUD state API**

Add to `ADispatchPlayerController`:

```cpp
UFUNCTION(BlueprintPure, Category = "HUD")
bool IsPlayerHUDRequestedVisible() const { return bPlayerHUDRequestedVisible; }
```

- [ ] **Step 5: Implement the notebook modal lifecycle**

Add `NativeConstruct` and `NativeDestruct` overrides. `NativeConstruct` calls `ActivateModalPresentation(Cast<ADispatchPlayerController>(GetOwningPlayer()))`; `NativeDestruct` calls `DeactivateModalPresentation()` before `Super::NativeDestruct()`.

`ActivateModalPresentation` must guard repeated activation, capture `IsPlayerHUDRequestedVisible`, hide the HUD, and call a private `EnsureModalBackground`.

`DeactivateModalPresentation` must restore the captured requested visibility only when a controller state was captured, then clear all modal-state flags and controller references.

- [ ] **Step 6: Implement the full-screen blur and tint**

`EnsureModalBackground` must:

```cpp
UCanvasPanel* RootCanvas = Cast<UCanvasPanel>(WidgetTree ? WidgetTree->RootWidget : nullptr);
UBackgroundBlur* Blur = WidgetTree->ConstructWidget<UBackgroundBlur>(
	UBackgroundBlur::StaticClass(),
	TEXT("NotebookModalBackgroundBlur"));
Blur->SetApplyAlphaToBlur(false);
Blur->SetOverrideAutoRadiusCalculation(true);
Blur->SetBlurRadius(8);
Blur->SetBlurStrength(12.0f);
Blur->SetVisibility(ESlateVisibility::HitTestInvisible);

UBorder* Tint = WidgetTree->ConstructWidget<UBorder>(
	UBorder::StaticClass(),
	TEXT("NotebookModalBackgroundTint"));
Tint->SetBrushColor(FLinearColor(0.0f, 0.0f, 0.0f, 0.20f));
Tint->SetVisibility(ESlateVisibility::HitTestInvisible);
Blur->SetContent(Tint);

UCanvasPanelSlot* Slot = RootCanvas->AddChildToCanvas(Blur);
Slot->SetAnchors(FAnchors(0.0f, 0.0f, 1.0f, 1.0f));
Slot->SetOffsets(FMargin(0.0f));
Slot->SetAlignment(FVector2D::ZeroVector);
Slot->SetAutoSize(false);
Slot->SetZOrder(-100);
```

If the root is not a canvas, log one warning and leave the existing hierarchy untouched.

- [ ] **Step 7: Build and verify GREEN**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Widget.QuestNotebook.ModalPresentation;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: build exit code `0`; modal presentation test passes with zero errors.

- [ ] **Step 8: Run the notebook regression group**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Widget.QuestNotebook;Quit' -unattended -nop4 -nullrhi -nosplash -log
```

Expected: all notebook tests pass.

- [ ] **Step 9: Review and commit only modal-presentation files**

Confirm `git diff` contains the intended native modal behavior and the new test while all pre-existing user changes remain unstaged. Commit only:

```powershell
git add -- Source/Dispatch/Public/Controller/DispatchPlayerController.h Source/Dispatch/Public/Widget/PlayerNotebookWidget.h Source/Dispatch/Private/Widget/PlayerNotebookWidget.cpp Source/Dispatch/Private/Tests/PlayerNotebookModalTests.cpp
git commit -m "feat: make player notebook modal"
```

