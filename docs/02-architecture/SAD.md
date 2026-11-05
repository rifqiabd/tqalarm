# Software Architecture Document

## 1. Purpose

TQAlarm is an offline-first school bell application for Indonesian schools.
The product starts as a browser-only Progressive Web App (PWA).
The product can later use a desktop bridge for system-level reliability.

This document maps the system structure for v1 and v1.5.
It does not define implementation code.

## 2. Scope by Stage

| Stage | Status | Main runtime | Data source | Bell engine |
|---|---|---|---|---|
| v1 | In scope | Browser PWA | IndexedDB | Web Audio API and Web Worker |
| v1.5 | In scope for bridge contract | Browser PWA plus Tauri bridge | Bridge SQLite when connected | Native audio and native timer |
| v2 | Out of scope | PWA route expansion | Supabase is optional | Not defined here |

The v1 PWA must work without a backend.
The v1.5 bridge must preserve all v1 features.

## 3. Architecture Principles

1. Client-side first is the primary architecture.
2. v1 uses zero backend services.
3. Offline use is a product requirement.
4. IndexedDB is the browser fallback store.
5. The bridge is the source of truth when it is connected.
6. The PWA and bridge communicate through a localhost WebSocket.
7. The WebSocket uses token authentication.
8. Kiosk and display are PWA routes.
9. The `/kiosk` route is part of v1.5.
10. The `/display` route is reserved for v2.
11. The system uses a default tone when custom audio fails.
12. The browser uses leader election to prevent duplicate bells.

## 4. System Context

```text
+------------------------------+
| School operator or teacher   |
+---------------+--------------+
                |
                v
+----------------------------------------------+
| TQAlarm PWA                                  |
| / dashboard                                  |
| /settings                                    |
| /onboarding                                  |
| /kiosk                                       |
| /display (v2)                                |
+----------------+-----------------------------+
                 |
                 | v1: local browser storage
                 | v1.5: localhost WebSocket
                 v
+----------------------------------------------+
| Browser runtime or Tauri bridge              |
|                                              |
| v1 browser:                                  |
| - IndexedDB                                  |
| - Web Audio API                              |
| - Web Worker and leader election             |
|                                              |
| v1.5 bridge:                                 |
| - SQLite source of truth                     |
| - Local MP3 filesystem                       |
| - Native audio                               |
| - Native timer                               |
| - Watchdog and fail-safe                     |
+----------------+-----------------------------+
                 |
                 | optional in later stages
                 v
+----------------------------------------------+
| Supabase cloud                               |
| v2 or later only                             |
| Sync and backup                              |
+----------------------------------------------+
```

## 5. PWA Components

### 5.1 Routes

| Route | Stage | Responsibility |
|---|---|---|
| `/` | v1 | Show and manage schedules and bells |
| `/settings` | v1 | Manage application settings |
| `/onboarding` | v1 | Guide setup and provide Test Bell |
| `/kiosk` | v1.5 | Provide kiosk operation through the PWA |
| `/display` | v2 | Provide the display board route |

Kiosk and display are routes.
They are not separate applications.

### 5.2 Browser data layer

The browser data layer stores schedules, bells, overrides, sounds, settings, logs, and presets in IndexedDB.
The data layer supports JSON export and import.
The data layer supports Excel import through SheetJS.
The data layer keeps future fields in the settings record.

### 5.3 Browser audio layer

The browser audio layer uses the Web Audio API.
The user must unlock audio through the onboarding or audio banner flow.
The audio layer plays a default tone when custom audio fails.
The audio layer supports the basic white, pink, and brown noise generator.

### 5.4 Browser scheduler

The browser scheduler uses a Web Worker.
Leader election selects one active browser tab for bell execution.
Non-leader tabs do not execute the same bell.

## 6. Bridge Components

The bridge is a Tauri desktop application.
The bridge accepts authenticated WebSocket requests from the PWA.
The bridge stores settings and supported records in SQLite.
The bridge stores MP3 files in the local filesystem.
The bridge uses native audio playback.
The bridge uses a native timer.
The bridge can continue bell execution when the browser closes.
The bridge provides auto-start, system tray, watchdog, fail-safe behavior, offline license validation, and auto-update in v1.5.

The bridge owns the active data when the PWA is connected.
The PWA uses IndexedDB when the bridge is not connected.

## 7. Data and Control Flow

```text
+---------+       local data        +----------------+
| PWA v1  | <----------------------> | IndexedDB      |
+----+----+                          +----------------+
     |
     | authenticated WebSocket
     v
+----+-------------------+
| Bridge v1.5            |
| SQLite and filesystem  |
+----+-------------------+
     |
     +--> native timer --> native audio --> bell
```

In v1, the PWA reads and writes IndexedDB.
In v1.5, the PWA sends bridge operations through the WebSocket.
The bridge reads and writes SQLite and the local filesystem.
The bridge executes the native timer and native audio path.
The PWA uses IndexedDB if the bridge connection is unavailable.

## 8. Reliability and Failure Behavior

The PWA must continue to work without cloud access.
The PWA must use a default tone when custom audio is not usable.
Leader election must prevent duplicate bell execution across browser tabs.
The bridge must provide watchdog and fail-safe behavior in v1.5.
The browser must fall back to IndexedDB when the bridge is not connected.

## 9. Technology Decisions

| Area | Technology |
|---|---|
| Frontend | Svelte 5 and SvelteKit |
| Language | TypeScript |
| PWA | Manifest and Service Worker |
| Browser audio | Web Audio API |
| Browser timer | Web Worker and leader election |
| Browser storage | IndexedDB |
| Bridge | Tauri |
| Bridge storage | SQLite |
| Bridge transport | Localhost WebSocket with token auth |
| Excel import | SheetJS |
| Internationalization | i18next |
| Cloud in later stages | Supabase |

## 10. Explicit Exclusions

This architecture does not include a v1 backend.
This architecture does not include automatic payment.
This architecture does not include the v1 display board user interface.
This architecture does not include academic calendar integration.
This architecture does not include complete English localization.
This architecture does not include Google Drive backup.
This architecture does not include social noise preset sharing.
