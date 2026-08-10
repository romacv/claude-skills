---
name: apple
description: Use when writing, editing, or reviewing Apple-platform code — iOS, macOS, watchOS, Swift, SwiftUI, Xcode projects, or Apple frameworks (SwiftData, Core Data, GRDB, SQLite, CloudKit, Photos, Foundation Models, Liquid Glass).
---

# Platform: Apple (iOS / macOS / watchOS)

## Overview
An opinionated house style for Apple-platform code. Load alongside the public Apple stack — Axiom plugin (`axiom-*` skills · `/axiom:*` · `xclog`/`xcsym`/`xcui` CLI), sosumi MCP (Apple docs). These conventions are the judgment layer the public tooling does not carry.

## Tools
- Use Axiom skills for all Apple platform work; drive simulators with `xcrun simctl`. Never install or use a build MCP — it sets its own build directory, which means a cold cache and a full rebuild every run.
- NEVER use mobile-mcp for any Apple target.

## Code
- Language: Swift 6 strict concurrency throughout.
- UI framework: SwiftUI only.
- Async: Swift Concurrency (async/await and AsyncSequence). Combine is legacy, do not write new code with it.
- State: `@Observable` only. No `ObservableObject`, no `@Published`.
- Persistence: SwiftData, Core Data, GRDB, or SQLite.
- Testing: Swift Testing framework. No XCTest unless the target requires it.
- One type per file. No multiple View declarations in one file. No DTO dumps.
- All public types must conform to `Sendable`.
- Actors own all mutable shared state.

## Simplify · adopt new · drop legacy
Core stance: push every cross-cutting concern to **one seam**, modularize **for the compiler**, and treat Combine / `@escaping` completion handlers / `ObservableObject` / UIKit-for-new-screens / Core-Data-for-greenfield / XCTest as **legacy bridges to retire, not patterns to extend**. Keep what's load-bearing (a working CloudKit-synced store with tested migrations); modernize what's greenfield. Make wrong states fail at launch/in tests, never in prod.

### DO
- **Modularize for the compiler, not the file browser:** split into many small SPM library targets layered by dependency direction (Foundation → Services → UI → Features), each with its own `Sources/` + `Tests/`. Folder-only "modules" inside one target enforce nothing.
- **Adopt Swift 6 strict concurrency incrementally:** opt in per target (shared `swiftSettings` + upcoming-feature flags) and migrate module-by-module, not the whole app at once — keep adoption visible and reversible per module.
- **One seam per cross-cutting concern — change once, propagate everywhere:**
  - *Platform types:* one type-alias file maps per-OS types to shared aliases. Example:
    ```swift
    #if os(macOS)
    import AppKit
    public typealias PlatformColor = NSColor
    public typealias PlatformView = NSView
    public typealias PlatformViewController = NSViewController
    public typealias PlatformImage = NSImage
    #elseif os(iOS)
    import UIKit
    public typealias PlatformColor = UIColor
    public typealias PlatformView = UIView
    public typealias PlatformViewController = UIViewController
    public typealias PlatformImage = UIImage
    #endif
    ```
  - *Design tokens:* one file of semantic color/font/icon tokens, each resolving its light/dark + per-platform value **inside** the token — never a hardcoded value at the call site.
  - *Dependency injection:* a lightweight, compile-time-checked DI using keyPaths (e.g., `@Injected(\.service)`). Inject protocols / composed existentials (`any Service`), never concretes. Example:
    ```swift
    public protocol DependencyKey {
        associatedtype Value
        static var currentValue: Value { get set }
    }
    public struct DependencyValues {
        private static var current = DependencyValues()
        public static subscript<K>(key: K.Type) -> K.Value where K: DependencyKey {
            get { key.currentValue }
            set { key.currentValue = newValue }
        }
        public static subscript<T>(_ keyPath: WritableKeyPath<DependencyValues, T>) -> T {
            get { current[keyPath: keyPath] }
            set { current[keyPath: keyPath] = newValue }
        }
    }
    @propertyWrapper
    public struct Injected<T> {
        private let keyPath: WritableKeyPath<DependencyValues, T>
        public var wrappedValue: T {
            get { DependencyValues[keyPath] }
            set { DependencyValues[keyPath] = newValue }
        }
        public init(_ keyPath: WritableKeyPath<DependencyValues, T>) {
            self.keyPath = keyPath
        }
    }
    ```
  - *Strings:* typed config (typed key + default) instead of raw `UserDefaults.standard.string(forKey:)`, and typed event factories instead of inline analytics strings.
- **Make wrong states unrepresentable early:** trap on invalid config defaults at construction so they fail at launch / in tests, not in prod.
- **Strict-concurrency escape hatches are audited exceptions, not defaults:** `@preconcurrency import` for legacy SDKs; `nonisolated(unsafe)` / `@unchecked Sendable` only where an invariant is hand-proven (a type that internally serializes access). Reach for an actor or an immutable `Sendable` value first.
- **Migrations:** versioned models + a domain-split migration manager (one file per migrated concern) + dedicated migration tests against an in-memory/temp store.

### DON'T → use instead
- `ObservableObject` + `@Published` + `@StateObject` → `@Observable` macro + plain `@State`.
- Combine for one-shot async or as an event bus (`AnyPublisher`, `Set<AnyCancellable>`) → `async/await`, `AsyncStream`/`AsyncSequence`, `Observation`.
- `@escaping` completion handlers in new APIs → `async` funcs; bridge legacy callbacks with `withCheckedContinuation`.
- UIKit/AppKit VC stacks, storyboards/nibs, manual Auto Layout for new screens → SwiftUI + `NavigationStack` (value-based routes); keep a VC router only as a legacy bridge.
- Raw `#if os()` sprinkled through feature code → the platform-type-alias seam + `Shared`/`iOS`/`macOS` folder split.
- New Core Data schemas → SwiftData (or GRDB for fine-grained SQL/perf). Keep Core Data + `NSPersistentCloudKitContainer` only where a synced store + migration history already exist — don't rewrite a working synced store for novelty.
- New XCTest suites → Swift Testing (`@Suite`/`@Test`/`#expect`/`#require`, `.serialized`, `@MainActor`); migrate older suites opportunistically.

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
- Verify to a **green build only** (+ existing fast Swift Testing unit tests) — do NOT launch/run the app, drive the simulator, capture screenshots, or use xcui / Axiom UI automation as a dev-loop step. Report build-green + a short what-to-test list, hand the running app to the maintainer for manual testing. Drive/capture the sim only when explicitly asks.
- Call `session_set_defaults` before first build in a session.
- Build target: latest booted simulator or nearest iPhone Pro with `useLatestOS true` (running/UI only on explicit request).
- Reuse one simulator; kill stale simulators if more than three are running.

## Platform divergence (DRY/KISS — no inline `#if os()` in view bodies)
- `#if os()` or `#if canImport()` inside a `var body` or any view builder = mandatory rewrite before proceeding.
- Isolate platform branches using exactly one of these three patterns:
  1. File-level guard: `#if os(macOS)` wraps the entire file — use for platform-exclusive features.
  2. `ViewModifier`: one modifier owns the `#if`; the call site contains no conditional — e.g. `.pageTabStyle(animating:)`.
  3. `View` extension: `func iOSOnly() -> some View` with the `#if` inside the extension body.
- Conditional modifiers or extensions isolate the branches. Example:
  ```swift
  extension View {
      @ViewBuilder
      public func modifyForPlatform<T: View>(@ViewBuilder _ transform: (Self) -> T) -> some View {
          transform(self)
      }
      
      public func platformBackground(_ color: Color) -> some View {
          #if os(macOS)
          return self.background(color)
          #else
          return self.background(color.ignoresSafeArea())
          #endif
      }
  }
  ```
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
