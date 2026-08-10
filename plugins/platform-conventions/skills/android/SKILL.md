---
name: android
description: Use when writing, editing, or reviewing native Android (Kotlin, Jetpack Compose) code, including emulator runtime, Material3, and design-token conventions.
---

# Platform: Android (Native)

## Overview
An opinionated house style for native Android. Load alongside the public stack — mobile-mcp (Android). These conventions are the judgment layer the public tooling does not carry.

## Tools
- Use mobile-mcp for all Android work.

## Code
- Language: Kotlin only. No Java new code.
- UI framework: Jetpack Compose. No XML layouts.
- Async: Coroutines and Flow. No RxJava.
- Design system: Material3.
- Architecture: ViewModel per screen. No logic in Composable functions.
- One type per file. No file-scope top-level Composables. No DTO dumps.

## UI tokens
Base rule: no hardcoded color/font/number/radius at a usage site — always a named token; add one if none fits. Recurring Composable recipe (2nd time) → shared Composable in `ui/components/`.
Android names: color `MaterialTheme.colorScheme.*` · font `AppTypography.*` · spacing/size & radius `Dimens.*`. No inline `TextStyle(...)`, color literal, or numeric `dp`/`sp` at a usage site.

## Runtime
- Verify to a **green build** (+ existing fast unit tests) — the autonomous ceiling. Report build-green + a short what-to-test list, then hand the running app to the maintainer for manual testing. Do NOT auto-run the app, drive the emulator, or screenshot as a dev-loop step.
- Run/drive only when explicitly asked: a single visible emulator (never headless, never more than one), native pixel coordinates for UI automation via mobile-mcp.
