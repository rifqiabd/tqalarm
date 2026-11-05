# Data Model

## 1. Purpose

This document defines the records that TQAlarm uses.
The model supports the browser PWA in v1.
The model supports the bridge in v1.5.

The browser stores records in IndexedDB.
The bridge stores records in SQLite when it is connected.
The bridge is the source of truth when it is connected.

## 2. Model Rules

1. Every record has a stable `id`.
2. Every record has `createdAt` and `updatedAt` timestamps.
3. Timestamps use ISO 8601 UTC strings.
4. The browser stores supported records in IndexedDB.
5. The bridge stores supported records in SQLite.
6. The bridge keeps the same logical field names as the browser.
7. The model keeps `announcements`, `schoolLogo`, and `displayMode`.
8. The model does not require a cloud service in v1.
9. The model does not define automatic payment data.

## 3. Record Map

```text
+-----------+       1 to many       +-------+
| schedules | --------------------> | bells |
+-----+-----+                        +---+---+
      ^                                 |
      |                                 | many to one
      |                                 v
+-----+-------+                      +--+------+
| overrides  | -------------------->| sounds  |
+------------+   bellIds[]          +---------+

+----------+       independent records       +---------+
| settings |                                | presets |
+----------+                                +---------+

+--------+
| logs   |
+--------+
```

An override references bell IDs through `bellIds`.
A bell references one sound through `soundId`.
A schedule contains many bells.

## 4. Common Field Rules

| Field | Type | Rule |
|---|---|---|
| `id` | string | Unique record identifier |
| `createdAt` | string | ISO 8601 UTC creation time |
| `updatedAt` | string | ISO 8601 UTC update time |

The application must preserve unknown supported fields during import and export.
The application must not remove future settings fields because the current UI does not use them.

## 5. Core Records

### 5.1 `schedules`

| Field | Type | Required | Description | Stage |
|---|---|---:|---|---|
| `id` | string | Yes | Schedule identifier | v1 |
| `name` | string | Yes | Schedule name | v1 |
| `isActive` | boolean | Yes | Schedule state | v1 |
| `days` | string[] | Yes | Active weekday values | v1 |
| `createdAt` | string | Yes | Creation time | v1 |
| `updatedAt` | string | Yes | Update time | v1 |

`days` must contain one or more supported weekday values.

### 5.2 `bells`

| Field | Type | Required | Description | Stage |
|---|---|---:|---|---|
| `id` | string | Yes | Bell identifier | v1 |
| `scheduleId` | string | Yes | Parent schedule identifier | v1 |
| `time` | string | Yes | Local clock time | v1 |
| `label` | string | Yes | Bell label | v1 |
| `soundId` | string | Yes | Sound identifier | v1 |
| `durationSec` | number | Yes | Bell duration in seconds | v1 |
| `preBellMin` | number | Yes | Minutes before the bell | v1 |
| `isActive` | boolean | Yes | Bell state | v1 |
| `createdAt` | string | Yes | Creation time | v1 |
| `updatedAt` | string | Yes | Update time | v1 |

`time` uses the school device local time.
`durationSec` must be zero or greater.
`preBellMin` must be zero or greater.

### 5.3 `overrides`

| Field | Type | Required | Description | Stage |
|---|---|---:|---|---|
| `id` | string | Yes | Override identifier | v1 |
| `date` | string | Yes | Local calendar date | v1 |
| `type` | string | Yes | `holiday`, `exam`, `event`, or `custom` | v1 |
| `bellIds` | string[] | Yes | Bell identifiers affected by the override | v1 |
| `note` | string | No | Operator note | v1 |
| `createdAt` | string | Yes | Creation time | v1 |
| `updatedAt` | string | Yes | Update time | v1 |

`date` uses the `YYYY-MM-DD` format.
`type` must use one value from the listed set.

### 5.4 `sounds`

| Field | Type | Required | Description | Stage |
|---|---|---:|---|---|
| `id` | string | Yes | Sound identifier | v1 |
| `name` | string | Yes | Sound name | v1 |
| `source` | string | Yes | `builtin` or `user` | v1 |
| `fileBlob` | binary or null | Conditional | Browser audio data | v1 |
| `filePath` | string or null | Conditional | Bridge filesystem path | v1.5 |
| `durationSec` | number | Yes | Sound duration in seconds | v1 |
| `createdAt` | string | Yes | Creation time | v1 |
| `updatedAt` | string | Yes | Update time | v1 |

A user sound uses `fileBlob` in the browser.
A bridge sound uses `filePath` in the bridge.
The default tone remains available as a fallback.

## 6. Settings Record

The `settings` record contains one application settings object.

| Field | Type | Required | Description | Stage |
|---|---|---:|---|---|
| `schoolName` | string | No | School name | v1 |
| `schoolLogo` | string or null | No | Future school logo reference | Preserved for v2 |
| `themeColor` | string | No | Theme color value | v1 |
| `locale` | string | Yes | Current language locale | v1 |
| `displayMode` | string or null | No | Future display mode value | Preserved for v2 |
| `announcements` | object[] | No | Future announcement records | Preserved for v2 |
| `audioUnlocked` | boolean | Yes | Browser audio unlock state | v1 |
| `bridgeConnected` | boolean | Yes | Current bridge connection state | v1.5 |
| `bridgeToken` | string or null | No | Local bridge authentication token | v1.5 |
| `createdAt` | string | Yes | Creation time | v1 |
| `updatedAt` | string | Yes | Update time | v1 |

The UI does not need to implement future announcement behavior in v1.
The data layer must preserve `announcements` during export and import.
The UI does not need to implement `schoolLogo` behavior in v1.
The data layer must preserve `schoolLogo` during export and import.
The UI does not need to implement `displayMode` behavior in v1.
The data layer must preserve `displayMode` during export and import.

## 7. Operational Records

### 7.1 `logs`

| Field | Type | Required | Description | Stage |
|---|---|---:|---|---|
| `id` | string | Yes | Log identifier | v1 |
| `timestamp` | string | Yes | Event time | v1 |
| `bellId` | string or null | No | Related bell identifier | v1 |
| `status` | string | Yes | Bell execution status | v1 |
| `errorMessage` | string or null | No | Error detail | v1 |
| `createdAt` | string | Yes | Creation time | v1 |
| `updatedAt` | string | Yes | Update time | v1 |

### 7.2 `presets`

| Field | Type | Required | Description | Stage |
|---|---|---:|---|---|
| `id` | string | Yes | Preset identifier | v1 |
| `name` | string | Yes | Preset name | v1 |
| `layers` | object[] | Yes | Noise layer configuration | v1 |
| `volume` | number | Yes | Preset volume | v1 |
| `isUserPreset` | boolean | Yes | User preset state | v1 |
| `createdAt` | string | Yes | Creation time | v1 |
| `updatedAt` | string | Yes | Update time | v1 |

`layers` supports the basic white, pink, and brown noise generator.

## 8. Storage by Stage

| Record | Browser v1 | Bridge v1.5 |
|---|---|---|
| `schedules` | IndexedDB | SQLite |
| `bells` | IndexedDB | SQLite |
| `overrides` | IndexedDB | SQLite |
| `sounds` | IndexedDB with `fileBlob` | SQLite metadata with filesystem `filePath` |
| `settings` | IndexedDB | SQLite |
| `logs` | IndexedDB | SQLite |
| `presets` | IndexedDB | SQLite |

The bridge may receive browser data through the authenticated API contract.
The bridge must keep the bridge copy as the active source of truth when connected.

## 9. Import and Export Boundaries

JSON export includes all supported records.
JSON import restores all supported records.
Excel import creates schedule and bell data from supported spreadsheet input.
Excel import does not define bridge synchronization.
The import process must preserve `announcements`, `schoolLogo`, and `displayMode` when JSON contains them.
