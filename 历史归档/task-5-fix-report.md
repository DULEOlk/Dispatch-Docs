# Task 5 Fix Report

## Scope

- Base: `7898001a7f7ebef366b6b9bf266b5bda01a84cd4`
- Commit message: `feat: add unified Story UI clue preview`
- Tracked scope: the eight Task 5 brief files only
- Excluded and untouched: coordinator/runtime/assets/config/`.claude`

## Review findings fixed

1. `FStoryWorkspaceEditorSurface` now keeps `ValidationWidget` as a weak pointer. The spawned `SDockTab` is the strong owner, so closing the production Validation tab destroys the widget, removes both delegates, and executes its `StopPreview` cleanup path.
2. `RefreshRequestValidation` always builds the current request and calls `FStoryCluePreviewRequest::Validate`, including incomplete selections. Hand-written early-return validation was removed.
3. Catalog diagnostics remain in `PreviewValidationErrors` for display, while `bCurrentRequestValid` alone records authoritative request validity. Start gating combines this validity with coordinator presence/state, so an unused invalid catalog entry is visible without blocking a valid selected request.
4. Task 5 Slate tests no longer call `StartCluePreview` with a valid request. UI states use the pure state callback seam, and panel/toolkit shutdown tests use a stopped coordinator lifecycle. Task 4 remains the owner of live PIE coverage.
5. Catalog refresh tests assert that the selected Day and still-registered Clue A survive while removed Clue B is pruned.
6. Exact C++ defaults are exposed through static default-path helpers shared by the constructor and tests, avoiding dependence on a user-overridden settings CDO.

## TDD evidence

- RED: `PureTestSeam` failed four tab-close weak-expiry/delegate assertions.
- RED: `ValidationPreview` failed two incomplete-selection authoritative Widget/map error assertions.
- RED: `ValidationPreview` failed because an unused invalid Catalog Inference diagnostic blocked a valid selected request.
- RED: static default helper calls failed compilation until the helpers were implemented.
- GREEN: `Dispatch.Editor.StoryWorkspace.Editor` passed all five tests after the minimal fixes.

## Final verification

- `Build.bat DispatchEditor Win64 Development ...`: succeeded.
- `Automation RunTests Dispatch.Editor.StoryWorkspace`: 38 completed, 0 failed.
- `Automation RunTests Dispatch.Widget.PlayerNotebook.CluePage`: 15 completed, 0 failed.
- `Automation RunTests Dispatch`: 238 completed, 0 failed.
- `git diff --check`: clean (line-ending conversion warnings only).
- `git diff --name-only`: exactly the eight Task 5 brief files.

## Commit

- `020e1b2` — `feat: add unified Story UI clue preview`
