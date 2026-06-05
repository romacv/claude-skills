---
name: frontend
description: Use when writing, editing, or reviewing web frontend code — Next.js App Router, TypeScript, React, shadcn/ui, Tailwind, or Cloudflare Pages.
---

# Platform: Frontend (Web)

## Overview
An opinionated house style for web frontend. Load alongside the public stack — `frontend-design` skill + claude-in-chrome MCP (only on explicit request to drive the UI). These conventions are the judgment layer the public tooling does not carry.

## Tools
- Use the frontend-design skill for all web UI work.
- Drive the browser (claude-in-chrome) only when explicitly asked — then visible, never headless. Not a dev-loop verification step.

## Code
- Framework: Next.js with App Router. No Pages Router for new code.
- Language: TypeScript. No plain JavaScript for new files.
- UI library: React with shadcn/ui components.
- Styling: Tailwind CSS only. No inline styles, no CSS-in-JS, no separate `.css` files unless required by a third-party library.
- Deployment target: Cloudflare Pages.
- One component per file. No multiple exported components from a single file.

## UI tokens
Base rule: no hardcoded color/font/number/radius at a usage site — always a named token; add one if none fits. Recurring JSX (2nd time) → shared component in `components/ui/`.
Web names: color/spacing/typography/radius via CSS custom properties or Tailwind theme tokens only — no hardcoded hex, `px`, `rem`, or `font-size` at a usage site.

## Runtime
- Verify to a **green build / typecheck / lint** (+ existing fast unit tests) — that is the autonomous ceiling. Report build-green + a short what-to-test list, then hand the running app to the maintainer for manual testing.
- Do NOT auto-run the dev server, click through the UI, or screenshot as a dev-loop step. Drive the browser (visible, never headless) only when explicitly asked.
