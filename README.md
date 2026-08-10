# claude-skills

Opinionated, cross-project platform **conventions** (house style) packaged as a Claude Code marketplace plugin.

## Plugin: `platform-conventions`

Per-domain `SKILL` files that load lazily (auto-matched by description) when a task hits that platform:

| Skill | Domain |
|---|---|
| `apple` | iOS / macOS / watchOS — Swift, SwiftUI, SwiftData/GRDB, CloudKit, Foundation Models |
| `backend` | Cloudflare Workers, Hono, TypeScript, Drizzle/D1 |
| `frontend` | Next.js App Router, TypeScript, React, shadcn/ui, Tailwind |
| `android` | Android (Kotlin/Compose) |
| `flutter` | Flutter (Dart/cross-platform) |

These are the *judgment layer* on top of public tooling (Axiom, the Cloudflare plugin, frontend-design, mobile-mcp) — naming, architecture, design-token and verification conventions the tooling itself does not enforce.

## Install

```
/plugin marketplace add romacv/claude-skills
/plugin install platform-conventions@claude-skills
```

## License

MIT — see [LICENSE](LICENSE).
