# API Contract

## 1. Purpose

This document defines the contracts for the TQAlarm PWA and bridge.
The contract covers PWA v1 and bridge v1.5 only.
The contract does not define cloud APIs.
The contract does not define automatic payment APIs.

## 2. Stage Status

| Contract area | Status | Stage |
|---|---|---|
| IndexedDB record access | Required | v1 |
| JSON export and import | Required | v1 |
| Excel import | Required | v1 |
| Browser bell execution | Required | v1 |
| Noise generator control | Required | v1 |
| Localhost WebSocket | Required | v1.5 |
| Token authentication | Required | v1.5 |
| Bridge data operations | Required | v1.5 |
| Bridge native bell execution | Required | v1.5 |
| Bridge file transfer | Required | v1.5 |
| Cloud synchronization | Out of scope | v2 |
| Automatic payment | Out of scope | Not defined |

## 3. Contract Rules

1. The PWA uses TypeScript data shapes for all local contracts.
2. The bridge uses the same logical field names as the PWA.
3. Each request has a unique `requestId`.
4. Each response returns the related `requestId`.
5. Each WebSocket message uses JSON unless this document defines binary data.
6. Each error returns a stable `code`.
7. The bridge rejects requests without valid token authentication.
8. The PWA uses IndexedDB when the bridge is not connected.
9. The bridge is the source of truth when the bridge is connected.

## 4. PWA v1 Local Contracts

### 4.1 IndexedDB stores

The PWA uses these IndexedDB stores:

- `schedules`
- `bells`
- `overrides`
- `sounds`
- `settings`
- `logs`
- `presets`

The PWA uses the data model for each store.
The PWA preserves `announcements`, `schoolLogo`, and `displayMode` in `settings`.

### 4.2 JSON export

The PWA exports all supported records as one JSON document.

```json
{
  "format": "tqalarm-json",
  "version": 1,
  "exportedAt": "2026-01-01T00:00:00.000Z",
  "data": {
    "schedules": [],
    "bells": [],
    "overrides": [],
    "sounds": [],
    "settings": {},
    "logs": [],
    "presets": []
  }
}
```

The PWA sets `format` to `tqalarm-json`.
The PWA sets `version` to `1` for the v1 format.
The PWA includes every supported store in `data`.
The PWA preserves future settings fields during export.

### 4.3 JSON import

The PWA accepts a JSON document with the export shape.
The PWA validates `format` before import.
The PWA validates `version` before import.
The PWA rejects unsupported versions.
The PWA preserves `announcements`, `schoolLogo`, and `displayMode`.
The PWA does not call the bridge for a browser-only import.

### 4.4 Excel import

The PWA accepts an Excel file through SheetJS.
The PWA creates schedule and bell records from supported columns.
The PWA validates each required value before it creates a record.
The PWA reports invalid rows to the user.
The Excel contract does not create bridge records directly.

### 4.5 Browser bell control

The PWA uses the browser scheduler for scheduled bells.
The PWA uses Web Worker timer control.
The PWA uses leader election before it executes a bell.
The PWA executes a bell only in the leader tab.
The PWA uses Web Audio API playback after audio unlock.
The PWA uses the default tone when custom audio fails.

The PWA exposes a Test Bell action during onboarding.
The Test Bell action uses the selected sound when the sound is usable.
The Test Bell action uses the default tone when the selected sound fails.

### 4.6 Noise generator control

The PWA supports these noise layers:

- `white`
- `pink`
- `brown`

The PWA stores user presets in the `presets` store.
The PWA does not share presets with another user in v1.

## 5. Bridge v1.5 WebSocket Contract

### 5.1 Connection

The PWA connects to a bridge WebSocket on localhost.
The bridge provides the configured localhost endpoint.
The PWA sends a token before it sends protected requests.
The bridge rejects protected requests before authentication succeeds.

The bridge sends this ready message after the connection opens:

```json
{
  "type": "bridge.ready",
  "requestId": "server-1",
  "version": "1.5",
  "authenticated": false
}
```

### 5.2 Authentication request

The PWA sends this message to authenticate:

```json
{
  "type": "auth.authenticate",
  "requestId": "req-1",
  "token": "local-token"
}
```

The bridge returns this success response:

```json
{
  "type": "auth.result",
  "requestId": "req-1",
  "ok": true,
  "bridgeVersion": "1.5"
}
```

The bridge returns this failure response for an invalid token:

```json
{
  "type": "auth.result",
  "requestId": "req-1",
  "ok": false,
  "error": {
    "code": "AUTH_INVALID_TOKEN",
    "message": "The token is not valid."
  }
}
```

### 5.3 Data read request

The PWA sends this message to read one store:

```json
{
  "type": "data.read",
  "requestId": "req-2",
  "store": "schedules"
}
```

The bridge returns this response:

```json
{
  "type": "data.result",
  "requestId": "req-2",
  "ok": true,
  "store": "schedules",
  "records": []
}
```

The `store` value must name a supported data store.
The bridge returns an error for an unsupported store.

### 5.4 Data write request

The PWA sends this message to replace records in one store:

```json
{
  "type": "data.write",
  "requestId": "req-3",
  "store": "schedules",
  "records": []
}
```

The bridge validates the records before it writes them.
The bridge writes valid records to SQLite.
The bridge returns the written records in the response.

```json
{
  "type": "data.result",
  "requestId": "req-3",
  "ok": true,
  "store": "schedules",
  "records": []
}
```

The bridge does not write invalid records.
The bridge returns `DATA_INVALID` for invalid records.

### 5.5 Settings synchronization

The PWA uses `data.read` to read bridge settings.
The PWA uses `data.write` to write bridge settings.
The bridge stores settings in SQLite.
The bridge remains the source of truth after a successful write.
The PWA updates its local fallback copy after a successful bridge response.

### 5.6 Sound metadata and file transfer

The PWA sends sound metadata through `data.write`.
The PWA sends MP3 content through a binary file message.
The binary message includes the related `requestId`.
The bridge stores the MP3 file in the local filesystem.
The bridge returns the resulting `filePath` in the response.

```json
{
  "type": "file.put",
  "requestId": "req-4",
  "soundId": "sound-1",
  "fileName": "bell.mp3",
  "contentType": "audio/mpeg",
  "contentLength": 123456
}
```

The PWA sends the binary content after the file header.
The bridge rejects content that does not match the declared length.
The bridge returns `filePath` after it stores the file.

### 5.7 Bell execution request

The PWA sends this message for a Test Bell action:

```json
{
  "type": "bell.test",
  "requestId": "req-5",
  "soundId": "sound-1",
  "durationSec": 3
}
```

The bridge uses native audio for the request.
The bridge returns the execution status.

```json
{
  "type": "bell.result",
  "requestId": "req-5",
  "ok": true,
  "status": "played"
}
```

The bridge uses `BELL_AUDIO_FAILED` when native audio fails.
The bridge uses the default tone when the selected sound fails.

### 5.8 Scheduler configuration

The PWA sends supported schedule, bell, and override records with `data.write`.
The bridge uses SQLite as the active data source after the write succeeds.
The bridge native timer uses the active bridge data.
The bridge can execute bells when the browser closes.

The bridge does not require a separate scheduler API in v1.5.

### 5.9 Error response

The bridge uses this error shape:

```json
{
  "type": "error",
  "requestId": "req-6",
  "ok": false,
  "error": {
    "code": "DATA_INVALID",
    "message": "The record is not valid."
  }
}
```

The bridge uses these error codes:

| Code | Meaning |
|---|---|
| `AUTH_REQUIRED` | The request needs authentication |
| `AUTH_INVALID_TOKEN` | The token is not valid |
| `DATA_INVALID` | The data does not match the model |
| `STORE_UNSUPPORTED` | The store is not supported |
| `FILE_INVALID` | The file header or content is not valid |
| `BELL_AUDIO_FAILED` | Native audio failed |
| `BRIDGE_INTERNAL` | The bridge could not complete the request |

## 6. Connection Failure Behavior

The PWA marks the bridge as disconnected after the WebSocket closes.
The PWA uses IndexedDB while the bridge is disconnected.
The PWA does not treat cloud access as a recovery requirement.
The PWA reports the bridge state through the settings state.
