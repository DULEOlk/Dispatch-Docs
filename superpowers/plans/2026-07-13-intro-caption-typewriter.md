# Intro Caption Typewriter Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make intro slide and camera-beat captions reveal one character at a time with a configurable short sound every two non-whitespace characters by default.

**Architecture:** Keep all reveal state and timer ownership inside `UIntroSlideshowWidget`; existing callers continue using `SetSlide` and `SetCaption`. A deterministic `UpdateTypewriterText` step performs one reveal unit, while the widget owns the repeating world timer and cancels it during replacement, clearing, or destruction.

**Tech Stack:** Unreal Engine 5.6 C++, UMG `UUserWidget`/`UTextBlock`, `FTimerManager`, `UGameplayStatics::PlaySound2D`, Unreal Editor automation tests.

## Global Constraints

- Defaults are exactly `0.04` seconds per character and one sound per `2` non-whitespace characters.
- Whitespace is displayed but neither advances nor independently triggers the sound cadence.
- Supplementary Unicode characters represented by a UTF-16 surrogate pair are revealed together; combining-mark grapheme clustering is out of scope.
- Existing slide sequencing, illustration transitions, camera movement, slide sounds, and beat sounds must not change.
- No Blueprint timer graph is required; all configurable properties remain editable in Widget Blueprint defaults.
- Replacing or clearing a caption and destroying the widget must invalidate the old timer.
- Preserve all unrelated dirty-worktree changes and stage only files named in each task.

---

## File Structure

- `Source/Dispatch/Public/Story/IntroSlideshowWidget.h`: public typewriter configuration/API plus private reveal state and timer lifecycle declarations.
- `Source/Dispatch/Private/Story/IntroSlideshowWidget.cpp`: caption initialization, one-step reveal, sound cadence, Unicode handling, and timer cleanup.
- `Source/Dispatch/Private/Tests/IntroSequenceTests.cpp`: reflection/default tests plus deterministic progressive reveal, replacement, whitespace, and cleanup coverage.

### Task 1: Reflected configuration and immediate fallback

**Files:**
- Modify: `Source/Dispatch/Public/Story/IntroSlideshowWidget.h`
- Modify: `Source/Dispatch/Private/Story/IntroSlideshowWidget.cpp`
- Test: `Source/Dispatch/Private/Tests/IntroSequenceTests.cpp`

**Interfaces:**
- Consumes: existing `void SetCaption(const FText& Caption)` and optional `Text_Caption` binding.
- Produces: public `bEnableTypewriter`, `TypewriterInterval`, `TypewriterSound`, and `SoundEveryCharacters` Blueprint defaults; safe empty/disabled/no-world behavior.

- [ ] **Step 1: Add failing reflected-default tests**

In `FIntroSlideshowWidgetBehaviorTest::RunTest`, immediately after the three `BindWidget` assertions, add:

```cpp
	const FBoolProperty* EnableTypewriterProperty = FindFProperty<FBoolProperty>(WidgetClass, TEXT("bEnableTypewriter"));
	const FFloatProperty* IntervalProperty = FindFProperty<FFloatProperty>(WidgetClass, TEXT("TypewriterInterval"));
	const FObjectProperty* TypewriterSoundProperty = FindFProperty<FObjectProperty>(WidgetClass, TEXT("TypewriterSound"));
	const FIntProperty* SoundCadenceProperty = FindFProperty<FIntProperty>(WidgetClass, TEXT("SoundEveryCharacters"));
	TestNotNull(TEXT("bEnableTypewriter is reflected"), EnableTypewriterProperty);
	TestNotNull(TEXT("TypewriterInterval is reflected"), IntervalProperty);
	TestTrue(TEXT("TypewriterSound stores a USoundBase"), TypewriterSoundProperty
		&& TypewriterSoundProperty->PropertyClass->IsChildOf(USoundBase::StaticClass()));
	TestNotNull(TEXT("SoundEveryCharacters is reflected"), SoundCadenceProperty);
	const UIntroSlideshowWidget* WidgetDefaults = GetDefault<UIntroSlideshowWidget>();
	TestTrue(TEXT("Typewriter is enabled by default"), WidgetDefaults->bEnableTypewriter);
	TestEqual(TEXT("Typewriter interval defaults to 0.04 seconds"), WidgetDefaults->TypewriterInterval, 0.04f);
	TestNull(TEXT("Typewriter sound defaults to null"), WidgetDefaults->TypewriterSound.Get());
	TestEqual(TEXT("Typewriter sound cadence defaults to two characters"), WidgetDefaults->SoundEveryCharacters, 2);
	if (IntervalProperty)
	{
		TestEqual(TEXT("Typewriter interval clamp is positive"), FCString::Atof(*IntervalProperty->GetMetaData(TEXT("ClampMin"))), 0.001f);
	}
	if (SoundCadenceProperty)
	{
		TestEqual(TEXT("Sound cadence clamp minimum is one"), FCString::Atoi(*SoundCadenceProperty->GetMetaData(TEXT("ClampMin"))), 1);
	}
```

After injecting `Caption`, set `Widget->bEnableTypewriter = false;` before the existing `SetSlide` checks, so those existing assertions continue to verify the immediate-display branch explicitly.

- [ ] **Step 2: Build to verify RED**

Run:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

Expected: compilation fails because the four new members do not exist on `UIntroSlideshowWidget`.

- [ ] **Step 3: Add the minimal public configuration**

In `IntroSlideshowWidget.h`, add `class USoundBase;` with the forward declarations and add this block in `public:` before the functions:

```cpp
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Story|Intro|Typewriter")
	bool bEnableTypewriter = true;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Story|Intro|Typewriter", meta = (ClampMin = "0.001", UIMin = "0.01"))
	float TypewriterInterval = 0.04f;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Story|Intro|Typewriter")
	TObjectPtr<USoundBase> TypewriterSound = nullptr;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Story|Intro|Typewriter", meta = (ClampMin = "1", UIMin = "1"))
	int32 SoundEveryCharacters = 2;
```

Do not add timer behavior in this task. The existing `SetCaption` remains the immediate fallback and already handles empty/unbound captions safely.

- [ ] **Step 4: Build and run the focused test to verify GREEN**

Run the build command from Step 2, then:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Story.Intro.Widget.Behavior;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: build exits `0`; the log reports `Dispatch.Story.Intro.Widget.Behavior` succeeded with zero errors.

- [ ] **Step 5: Commit the configuration slice**

```powershell
git add -- 'Source/Dispatch/Public/Story/IntroSlideshowWidget.h' 'Source/Dispatch/Private/Tests/IntroSequenceTests.cpp'
git commit -m "feat: configure intro caption typewriter"
```

### Task 2: Progressive reveal, sound cadence, and lifecycle cleanup

**Files:**
- Modify: `Source/Dispatch/Public/Story/IntroSlideshowWidget.h`
- Modify: `Source/Dispatch/Private/Story/IntroSlideshowWidget.cpp`
- Test: `Source/Dispatch/Private/Tests/IntroSequenceTests.cpp`

**Interfaces:**
- Consumes: Task 1's four public configuration properties and existing `SetCaption`/`ClearSlide` calls.
- Produces: `UFUNCTION(BlueprintCallable) void UpdateTypewriterText()`, timer-driven reveal, per-cadence sound playback, surrogate-pair-safe advancement, and cleanup from `ClearSlide`/`NativeDestruct`.

- [ ] **Step 1: Add deterministic failing progression tests**

Add these helpers inside `namespace Dispatch::IntroTests` after `AssignWidget`:

```cpp
static int32 GetWidgetIntState(const UIntroSlideshowWidget* Widget, const FName PropertyName)
{
	const FIntProperty* Property = FindFProperty<FIntProperty>(Widget->GetClass(), PropertyName);
	return Property ? Property->GetPropertyValue_InContainer(Widget) : INDEX_NONE;
}

static UIntroSlideshowWidget* CreateWorldWidget(FAutomationTestBase& Test)
{
	FAutomationEditorCommonUtils::CreateNewMap();
	UWorld* World = GEditor ? GEditor->GetEditorWorldContext().World() : nullptr;
	Test.TestNotNull(TEXT("Typewriter test world exists"), World);
	return World ? CreateWidget<UIntroSlideshowWidget>(World, UIntroSlideshowWidget::StaticClass()) : nullptr;
}
```

Add a new automation test after `FIntroSlideshowWidgetBehaviorTest`:

```cpp
IMPLEMENT_SIMPLE_AUTOMATION_TEST(
	FIntroSlideshowWidgetTypewriterTest,
	"Dispatch.Story.Intro.Widget.Typewriter",
	EAutomationTestFlags::EditorContext | EAutomationTestFlags::EngineFilter)

bool FIntroSlideshowWidgetTypewriterTest::RunTest(const FString& Parameters)
{
	UIntroSlideshowWidget* Widget = Dispatch::IntroTests::CreateWorldWidget(*this);
	TestNotNull(TEXT("World-backed intro widget exists"), Widget);
	if (!Widget)
	{
		return false;
	}

	UTextBlock* Caption = Dispatch::IntroTests::AssignWidget<UTextBlock>(Widget, TEXT("Text_Caption"));
	TestNotNull(TEXT("Caption can be injected"), Caption);
	if (!Caption)
	{
		return false;
	}

	Widget->TypewriterInterval = 10.0f;
	Widget->SoundEveryCharacters = 2;
	Widget->SetCaption(FText::FromString(TEXT("A B")));
	TestEqual(TEXT("SetCaption reveals the first character immediately"), Caption->GetText().ToString(), FString(TEXT("A")));
	TestEqual(TEXT("First visible character advances cadence"),
		Dispatch::IntroTests::GetWidgetIntState(Widget, TEXT("NonWhitespaceCharactersSinceSound")), 1);

	Widget->UpdateTypewriterText();
	TestEqual(TEXT("Whitespace is revealed"), Caption->GetText().ToString(), FString(TEXT("A ")));
	TestEqual(TEXT("Whitespace does not advance cadence"),
		Dispatch::IntroTests::GetWidgetIntState(Widget, TEXT("NonWhitespaceCharactersSinceSound")), 1);

	Widget->UpdateTypewriterText();
	TestEqual(TEXT("Next non-whitespace character is revealed"), Caption->GetText().ToString(), FString(TEXT("A B")));
	TestEqual(TEXT("Cadence resets after two non-whitespace characters"),
		Dispatch::IntroTests::GetWidgetIntState(Widget, TEXT("NonWhitespaceCharactersSinceSound")), 0);

	Widget->SetCaption(FText::FromString(TEXT("XYZ")));
	TestEqual(TEXT("Replacement caption restarts from its first character"), Caption->GetText().ToString(), FString(TEXT("X")));
	Widget->UpdateTypewriterText();
	TestEqual(TEXT("Replacement does not retain old caption text"), Caption->GetText().ToString(), FString(TEXT("XY")));

	Widget->SetCaption(FText::FromString(FString::Chr(0xD83D) + FString::Chr(0xDE80) + TEXT("!")));
	TestEqual(TEXT("A surrogate pair is revealed as one display unit"), Caption->GetText().ToString(), FString::Chr(0xD83D) + FString::Chr(0xDE80));

	Widget->ClearSlide();
	TestTrue(TEXT("ClearSlide removes typewriter text"), Caption->GetText().IsEmpty());
	Widget->UpdateTypewriterText();
	TestTrue(TEXT("A stale update cannot restore a cleared caption"), Caption->GetText().IsEmpty());
	return true;
}
```

Also add `#include "Blueprint/UserWidget.h"` and `#include "TimerManager.h"` only if compilation requires them for `CreateWidget` or timer assertions.

- [ ] **Step 2: Build to verify RED**

Run the Task 1 build command.

Expected: compilation fails because `UpdateTypewriterText` and `NonWhitespaceCharactersSinceSound` do not exist.

- [ ] **Step 3: Declare reveal API, state, and lifecycle**

In `IntroSlideshowWidget.h`, add `#include "TimerManager.h"` after `CoreMinimal.h`. Add this public function after `SetCaption`:

```cpp
	UFUNCTION(BlueprintCallable, Category = "Story|Intro|Typewriter")
	void UpdateTypewriterText();
```

At the start of `protected:`, add:

```cpp
	virtual void NativeDestruct() override;
```

After the bound-widget properties, add:

```cpp
private:
	void StopTypewriter();
	void ShowCaptionImmediately();

	FString FullCaptionString;
	FTimerHandle TypewriterTimerHandle;

	UPROPERTY(Transient)
	int32 CurrentCaptionIndex = 0;

	UPROPERTY(Transient)
	int32 NonWhitespaceCharactersSinceSound = 0;
```

- [ ] **Step 4: Implement minimal timer-driven behavior**

In `IntroSlideshowWidget.cpp`, add:

```cpp
#include "Kismet/GameplayStatics.h"
#include "Sound/SoundBase.h"
#include "TimerManager.h"
```

Replace `SetCaption` with:

```cpp
void UIntroSlideshowWidget::SetCaption(const FText& Caption)
{
	StopTypewriter();
	FullCaptionString = Caption.ToString();
	CurrentCaptionIndex = 0;
	NonWhitespaceCharactersSinceSound = 0;

	if (!IsValid(Text_Caption))
	{
		return;
	}

	if (FullCaptionString.IsEmpty())
	{
		Text_Caption->SetText(FText::GetEmpty());
		Text_Caption->SetVisibility(ESlateVisibility::Collapsed);
		return;
	}

	Text_Caption->SetVisibility(ESlateVisibility::HitTestInvisible);
	if (!bEnableTypewriter || TypewriterInterval <= 0.0f || !IsValid(GetWorld()))
	{
		ShowCaptionImmediately();
		return;
	}

	Text_Caption->SetText(FText::GetEmpty());
	UpdateTypewriterText();
	if (CurrentCaptionIndex < FullCaptionString.Len())
	{
		GetWorld()->GetTimerManager().SetTimer(
			TypewriterTimerHandle,
			this,
			&UIntroSlideshowWidget::UpdateTypewriterText,
			FMath::Max(0.001f, TypewriterInterval),
			true);
	}
}
```

Add the following methods before `SetIllustrationTransform`:

```cpp
void UIntroSlideshowWidget::UpdateTypewriterText()
{
	if (!IsValid(Text_Caption) || CurrentCaptionIndex >= FullCaptionString.Len())
	{
		StopTypewriter();
		return;
	}

	const int32 CharacterStart = CurrentCaptionIndex;
	++CurrentCaptionIndex;
	if (FChar::IsHighSurrogate(FullCaptionString[CharacterStart])
		&& CurrentCaptionIndex < FullCaptionString.Len()
		&& FChar::IsLowSurrogate(FullCaptionString[CurrentCaptionIndex]))
	{
		++CurrentCaptionIndex;
	}

	Text_Caption->SetText(FText::FromString(FullCaptionString.Left(CurrentCaptionIndex)));
	if (!FChar::IsWhitespace(FullCaptionString[CharacterStart]))
	{
		++NonWhitespaceCharactersSinceSound;
		if (NonWhitespaceCharactersSinceSound >= FMath::Max(1, SoundEveryCharacters))
		{
			NonWhitespaceCharactersSinceSound = 0;
			if (IsValid(TypewriterSound.Get()))
			{
				UGameplayStatics::PlaySound2D(this, TypewriterSound.Get());
			}
		}
	}

	if (CurrentCaptionIndex >= FullCaptionString.Len())
	{
		StopTypewriter();
	}
}

void UIntroSlideshowWidget::StopTypewriter()
{
	if (UWorld* World = GetWorld(); IsValid(World))
	{
		World->GetTimerManager().ClearTimer(TypewriterTimerHandle);
	}
	else
	{
		TypewriterTimerHandle.Invalidate();
	}
}

void UIntroSlideshowWidget::ShowCaptionImmediately()
{
	if (IsValid(Text_Caption))
	{
		Text_Caption->SetText(FText::FromString(FullCaptionString));
	}
	CurrentCaptionIndex = FullCaptionString.Len();
}

void UIntroSlideshowWidget::NativeDestruct()
{
	StopTypewriter();
	Super::NativeDestruct();
}
```

At the start of `ClearSlide`, before clearing the image, add:

```cpp
	StopTypewriter();
	FullCaptionString.Reset();
	CurrentCaptionIndex = 0;
	NonWhitespaceCharactersSinceSound = 0;
```

The existing `SetCaption(FText::GetEmpty())` later in `ClearSlide` remains; it is idempotent and owns caption visibility.

- [ ] **Step 5: Build and run focused tests to verify GREEN**

Run the Task 1 build command, then:

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Story.Intro.Widget;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: build exits `0`; both `Dispatch.Story.Intro.Widget.Behavior` and `Dispatch.Story.Intro.Widget.Typewriter` succeed with zero errors.

- [ ] **Step 6: Run the complete intro regression suite**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Story.Intro;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

Expected: all `Dispatch.Story.Intro` tests succeed with zero errors. Inspect `Saved/Logs/Dispatch.log` if the process exit code does not convey automation failures.

- [ ] **Step 7: Review the final diff and commit**

```powershell
git diff --check -- 'Source/Dispatch/Public/Story/IntroSlideshowWidget.h' 'Source/Dispatch/Private/Story/IntroSlideshowWidget.cpp' 'Source/Dispatch/Private/Tests/IntroSequenceTests.cpp'
git diff -- 'Source/Dispatch/Public/Story/IntroSlideshowWidget.h' 'Source/Dispatch/Private/Story/IntroSlideshowWidget.cpp' 'Source/Dispatch/Private/Tests/IntroSequenceTests.cpp'
git add -- 'Source/Dispatch/Public/Story/IntroSlideshowWidget.h' 'Source/Dispatch/Private/Story/IntroSlideshowWidget.cpp' 'Source/Dispatch/Private/Tests/IntroSequenceTests.cpp'
git commit -m "feat: animate intro captions with typewriter audio"
```

Expected: the diff contains only the approved typewriter feature and its tests; the commit succeeds without staging unrelated `.uasset` or documentation deletions.
