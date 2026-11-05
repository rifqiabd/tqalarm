# ADR 0002: Bridge Source of Truth

- **Status:** Accepted
- **Date:** 2026-11-05

## Context

The PWA stores data in IndexedDB in v1.
The bridge adds SQLite and filesystem storage in v1.5.
The bridge must keep bells running when the browser closes.

Two active data sources can create different schedule states.
The system needs one active source when the bridge is connected.

## Decision

The bridge is the source of truth when it is connected.
The bridge stores active settings and supported records in SQLite.
The bridge stores MP3 files in the local filesystem.
The bridge runs the native timer from bridge data.
The bridge runs native audio from bridge data.

The PWA uses IndexedDB as its local fallback.
The PWA updates its fallback copy after a successful bridge write.
The PWA uses the fallback copy when the bridge is not connected.

The PWA and bridge use a localhost WebSocket.
The WebSocket uses token authentication.

## Consequences

The bridge can run bells when the browser closes.
The bridge can use native timer behavior.
The bridge can use native audio behavior.
The system has one active data source when the bridge is connected.

The PWA must show bridge connection state.
The PWA must handle bridge connection failure.
The PWA must keep valid IndexedDB data for fallback use.

The bridge and PWA must use the same logical field names.

## Alternatives Rejected

### Browser as source of truth

This option would not support bridge-first execution when the browser closes.
It would not use the locked bridge source-of-truth decision.

### Cloud as source of truth

This option would require a network service.
It would conflict with offline-first operation and v1 scope.
