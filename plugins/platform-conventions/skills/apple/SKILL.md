---
name: apple
description: Use when writing, editing, or reviewing Apple-platform code — iOS, macOS, watchOS, Swift, SwiftUI, Xcode projects, or Apple frameworks (SwiftData, GRDB, CloudKit, Photos, Foundation Models, Liquid Glass).
---

# Platform: Apple (iOS / macOS / watchOS)

## Overview
An opinionated house style for Apple-platform code. Load alongside the public Apple stack — Axiom plugin (`axiom-*` skills · `/axiom:*` · `xclog`/`xcsym`/`xcui` CLI), XcodeBuildMCP/`xcodebuildmcp-cli`, sosumi MCP (Apple docs). These conventions are the judgment layer the public tooling does not carry.

## Tools
- Use Axiom skills and XcodeBuildMCP for all Apple platform work.
- NEVER use mobile-mcp for any Apple target.

## Code
- Language: Swift 6 strict concurrency throughout.
- UI framework: SwiftUI only. No UIKit or AppKit unless bridging is unavoidable.
- Async: async/await and AsyncSequence. No Combine.
- State: `@Observable` only. No `ObservableObject`, no `@Published`.
- Persistence: SwiftData or GRDB. No Core Data.
- Testing: Swift Testing framework. No XCTest unless the target requires it.
- One type per file. No multiple View declarations in one file. No DTO dumps.
- All public types must conform to `Sendable`.
- Actors own all mutable shared state.

## Constants & configuration (no magic strings)
A repeated string literal drifts silently; a value already in a config has two sources of truth; a hardcoded value blocks white-label reuse. So:
- Identifiers used in 2+ places (CloudKit zone/record-type names, `UserDefaults` keys, notification names) → one `static let` in a single config type. Never a bare literal at the call site.
- A value that exists in a config file is owned by that file — read it, don't re-type it:
  - CloudKit container → `CKContainer.default()` (resolves the first id in the entitlement). Never `CKContainer(identifier: "iCloud.…")`.
  - Bundle id, version, build-variant values → `Bundle.main` / Info.plist (`Bundle.main.bundleIdentifier`, `infoDictionary`), fed by `.xcconfig`.
  - Secrets/endpoints → env var or build setting, never a literal (see security rules).
- Per-environment or per-reseller values belong in `.xcconfig` + Info.plist, surfaced through one typed `AppConfig` accessor — no `#if DEBUG` string forks scattered in code.

## UI tokens
Base rule (all platforms): no hardcoded color, font, number, or radius at a call site — always a named token; if none fits, add one. A view recipe seen a 2nd time → shared component in `UI/Components/`.
Apple names: color `AppColor.*` · font `AppFont.*` · spacing/size `AppSpacing.*` · radius `AppShape.*` · icons `SFIcon.*`.
- Spacing/size: `AppSpacing.*` steps only — a numeric literal at a call site is a violation (`.padding(32)`, `.frame(width: 240)`, `cornerRadius(8)`). No step fits → add one.
- Icons: `SFIcon.*` — never `Image(systemName:)` at a call site; use `SFIcon.xxx.rawValue` for `systemImage:`. Repeated render recipe (`.symbolRenderingMode`, `.foregroundStyle`, `.font`) → a factory method on `SFIcon` (e.g. `.closeButtonView(...)`), one line at the site.
- Recurring view recipe (2nd time) → a `ViewModifier` + `View` extension.

## Runtime
- Verify to a **green build only** (+ existing fast Swift Testing unit tests) — do NOT launch/run the app, drive the simulator, capture screenshots, or use xcui / Axiom UI automation as a dev-loop step. Report build-green + a short what-to-test list, hand the running app to the maintainer for manual testing. Drive/capture the sim only when explicitly asked.
- Call `session_set_defaults` before first build in a session.
- Build target: latest booted simulator or nearest iPhone Pro with `useLatestOS true` (running/UI only on explicit request).
- Reuse one simulator; kill stale simulators if more than three are running.

## Platform divergence (DRY/KISS — no inline `#if os()` in view bodies)
- `#if os()` or `#if canImport()` inside a `var body` or any view builder = mandatory rewrite before proceeding.
- Isolate platform branches using exactly one of these three patterns:
  1. File-level guard: `#if os(macOS)` wraps the entire file — use for platform-exclusive features.
  2. `ViewModifier`: one modifier owns the `#if`; the call site contains no conditional — e.g. `.pageTabStyle(animating:)`.
  3. `View` extension: `func iOSOnly() -> some View` with the `#if` inside the extension body.
- Never duplicate a modifier call block across two `#if` branches — factor the shared block out first.
- A platform branch (`#if os()`, `#if canImport()`) that appears more than once → extract to a named modifier, extension, or type immediately.
- Same logic in two branches = wrong abstraction level; fix it.

## Concurrency and ObjC interop
- Non-Sendable ObjC objects (e.g. `NSMetadataItem`) must be converted to `Sendable` Swift value types before entering any `Task { }` closure.
- Never capture a non-Sendable ObjC reference across an async boundary.

## watchOS specifics
- watchOS targets ship in one of two models — pick deliberately, never by default:
  - **Independent** (`WKRunsIndependentlyOfCompanionApp = YES`): installs/updates on its own, separate App Store product. Use when the watch app is fully useful without the phone.
  - **Embedded iOS companion** (`WKRunsIndependentlyOfCompanionApp = NO`): bundled inside the iOS app and shipped through the single iOS record (TestFlight/App Store), so it reaches testers/users with the phone build. Requires: watch bundle id = `<iOS-id>.watchkitapp`, `WKCompanionAppBundleIdentifier` = the iOS bundle id, and the iOS target embedding it via a target dependency + "Embed Watch Content" copy-files phase. If the iOS target is multiplatform (`SDKROOT=auto`), that phase **and** the dependency must be `platformFilter = ios` or the macOS build fails trying to embed a watchOS app.
- No CKSyncEngine on watchOS — use `CKDatabase` directly for read-only CloudKit access.
- `PHAsset` and Photos framework are absent on watchOS; guard all Photos API behind `#if canImport(Photos)`.
- Background refresh via `backgroundTask(.appRefresh)` only — no persistent background processes.
