# TQAlarm Technical Setup

## Purpose

This document describes the locked setup boundaries for TQAlarm. It is a map for implementation, not an implementation guide with unapproved configuration values.

## Locked stack

| Area | Technology | v1 boundary |
|---|---|---|
| Frontend | Svelte 5 + SvelteKit | Browser PWA |
| Language | TypeScript | Application source |
| PWA | Manifest + Service Worker | Offline-first delivery |
| Browser audio | Web Audio API | Requires explicit audio unlock |
| Browser timer | Web Worker + Leader Election | Prevents duplicate bells across tabs |
| Browser storage | IndexedDB (`idb` or Dexie) | Local fallback and the v1 source for browser-only use |
| Excel import | SheetJS (`xlsx`) | Import only |
| Internationalization | `i18next` | Indonesian complete in v1; English structure ready |
| Bridge | Tauri | v1.5 desktop scope, not v1 |
| Bridge storage | SQLite | Bridge source of truth when connected in v1.5 |
| PWA-to-bridge transport | WebSocket on `localhost` with token authentication | v1.5 boundary; not a browser extension |
| Cloud | Supabase | Optional later capability; not required by v1 |
| Testing | Vitest + Playwright + `svelte-check` | Unit, browser, and type/component checks |

The repository does not yet define dependency versions, package-manager commands, environment-variable names, ports, WebSocket message schemas, token format, build profiles, or deployment-specific configuration. Do not infer or add those values here. Define them in the appropriate technical contract or implementation change after approval.

## Workspace boundary

TQAlarm has two related workspaces:

- **PWA workspace:** the SvelteKit application. It owns the dashboard, `/settings`, and `/onboarding` routes, browser audio, the Web Worker timer, leader election, IndexedDB persistence, JSON export/import, and SheetJS Excel import for v1. `/kiosk` remains a PWA route prepared for the bridge stage; `/display` is reserved for a later stage.
- **Bridge workspace:** the Tauri desktop application for v1.5. It owns native audio, native timing, SQLite settings, local MP3 filesystem storage, and continued operation when the browser is closed. It is not part of the v1 browser-only implementation.

When a bridge is connected in v1.5, it is the source of truth. When it is unavailable, the browser uses local IndexedDB as fallback. The PWA and bridge communicate only through authenticated `localhost` WebSocket transport, with binary file transfer where the approved contract requires it. Do not substitute an extension, daemon, agent, or service for the bridge.

## Storage model

The browser data model is represented in IndexedDB. The bridge data model is represented in SQLite, with MP3 files in the local filesystem. The core entities include schedules, bells, overrides, sounds, settings, and logs. Preserve the future fields `announcements`, `schoolLogo`, and `displayMode` even when their UI is not available.

The v1 application must remain client-side first, offline-first, and zero-backend. Supabase is optional future sync/backup infrastructure; it is not a v1 setup dependency.

## Local WebSocket boundary

The `localhost` WebSocket and token authentication belong to the PWA-to-bridge integration in v1.5. The current product decisions do not establish a port, endpoint path, handshake payload, token storage rule, reconnection policy, or message schema. Those details must remain unspecified until an approved API contract defines them.

## Testing tools

Use the locked testing tools as follows:

- **Vitest:** unit and focused module tests for browser logic such as schedule decisions, data transforms, and leader-election behavior.
- **Playwright:** browser end-to-end checks for onboarding, audio unlock, schedule operation, import/export, and offline browser flows.
- **`svelte-check`:** Svelte and TypeScript diagnostics for the PWA workspace.

Bridge-specific integration checks belong to the v1.5 boundary and must not be treated as v1 acceptance checks. This document does not prescribe unapproved CI commands, browser matrices, ports, or coverage thresholds.
