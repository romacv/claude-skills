---
name: android-flutter
description: Use when writing, editing, or reviewing Android (Kotlin, Jetpack Compose) or Flutter (Dart) code, including emulator/simulator runtime and design-token conventions.
---

# Platform: Android (and Flutter)

## Overview
An opinionated house style for Android and Flutter. Load alongside the public stack — mobile-mcp (Android) / XcodeBuildMCP (Flutter iOS target). These conventions are the judgment layer the public tooling does not carry.

## Android (Kotlin / Compose)

### Tools
- Use mobile-mcp for all Android work.
- Never use XcodeBuildMCP for Android targets.

### Code
- Language: Kotlin only. No Java new code.
- UI framework: Jetpack Compose. No XML layouts.
- Async: Coroutines and Flow. No RxJava.
- Design system: Material3.
- Architecture: ViewModel per screen. No logic in Composable functions.
- One type per file. No file-scope top-level Composables. No DTO dumps.

### UI tokens
Base rule: no hardcoded color/font/number/radius at a usage site — always a named token; add one if none fits. Recurring Composable recipe (2nd time) → shared Composable in `ui/components/`.
Android names: color `MaterialTheme.colorScheme.*` · font `AppTypography.*` · spacing/size & radius `Dimens.*`. No inline `TextStyle(...)`, color literal, or numeric `dp`/`sp` at a usage site.

### Runtime
- Verify to a **green build** (+ existing fast unit tests) — the autonomous ceiling. Report build-green + a short what-to-test list, then hand the running app to the maintainer for manual testing. Do NOT auto-run the app, drive the emulator, or screenshot as a dev-loop step.
- Run/drive only when explicitly asked: a single visible emulator (never headless, never more than one), native pixel coordinates for UI automation via mobile-mcp.

---

## Flutter (Dart / cross-platform mobile)

### Tools
- Android target: mobile-mcp.
- iOS target: XcodeBuildMCP. Never use mobile-mcp for the iOS Flutter target.
- No CocoaPods. iOS dependencies via SPM only.

### Code
- Language: Dart.
- State management: BLoC or Riverpod. No setState in feature screens.
- Navigation: go_router.
- Structure: feature-first folder layout.
- One type per file; file names in snake_case.
- No file-scope top-level widget functions outside `*/widgets/` or `lib/ui/components/`.

### UI tokens
Base rule as above. Recurring widget recipe (2nd time) → shared widget.
Flutter names: color `AppColors.*` · font `AppTextStyles.*` · spacing/size `AppSpacing.*` · radius `AppBorderRadius.*`. No inline `Color(...)`, `TextStyle(...)`, numeric padding/`SizedBox`, or `BorderRadius.circular(...)` at a usage site.

### Runtime
- Verify to a **green build** (+ existing fast unit tests), then hand the running app to the maintainer for manual testing — no autonomous app-run / UI automation as a dev-loop step.
- Run/drive only on explicit request: Android — single visible emulator via mobile-mcp; iOS — single booted simulator via XcodeBuildMCP with `useLatestOS true`.
