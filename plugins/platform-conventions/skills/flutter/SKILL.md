---
name: flutter
description: Use when writing, editing, or reviewing Flutter (Dart) cross-platform mobile code, including iOS/Android target runtimes, state management (BLoC/Riverpod), and design-token conventions.
---

# Platform: Flutter (Cross-platform)

## Overview
An opinionated house style for Flutter. Load alongside the public stack — mobile-mcp (Android) / XcodeBuildMCP (Flutter iOS target). These conventions are the judgment layer the public tooling does not carry.

## Tools
- Android target: mobile-mcp.
- iOS target: XcodeBuildMCP. Never use mobile-mcp for the iOS Flutter target.
- No CocoaPods. iOS dependencies via SPM only.

## Code
- Language: Dart.
- State management: BLoC or Riverpod. No setState in feature screens.
- Navigation: go_router.
- Structure: feature-first folder layout.
- One type per file; file names in snake_case.
- No file-scope top-level widget functions outside `*/widgets/` or `lib/ui/components/`.

## UI tokens
Base rule: no hardcoded color/font/number/radius at a usage site — always a named token; add one if none fits. Recurring widget recipe (2nd time) → shared widget.
Flutter names: color `AppColors.*` · font `AppTextStyles.*` · spacing/size `AppSpacing.*` · radius `AppBorderRadius.*`. No inline `Color(...)`, `TextStyle(...)`, numeric padding/`SizedBox`, or `BorderRadius.circular(...)` at a usage site.

## Runtime
- Verify to a **green build** (+ existing fast unit tests), then hand the running app to the maintainer for manual testing — no autonomous app-run / UI automation as a dev-loop step.
- Run/drive only on explicit request: Android — single visible emulator via mobile-mcp; iOS — single booted simulator via XcodeBuildMCP with `useLatestOS true`.
