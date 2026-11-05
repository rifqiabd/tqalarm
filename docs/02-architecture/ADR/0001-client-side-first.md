# ADR 0001: Client-Side First

- **Status:** Accepted
- **Date:** 2026-11-05

## Context

TQAlarm must work for schools without a backend.
The free version must provide useful schedule and bell features.
The product must support offline use.

A backend would add a required network dependency.
A backend would conflict with the zero-backend decision for v1.

## Decision

TQAlarm uses a client-side first architecture.
The v1 PWA stores application data in IndexedDB.
The v1 PWA uses Web Audio API for browser audio.
The v1 PWA uses a Web Worker for the browser timer.
The v1 PWA uses leader election to prevent duplicate bells.

The v1 PWA does not require a backend.
The v1 PWA does not require cloud access.

The bridge remains the planned v1.5 path for native reliability.
This decision does not remove any v1 feature from the PWA.

## Consequences

The PWA can work offline.
The PWA has no server dependency in v1.
The PWA must manage local storage and local data export.
The PWA must handle browser audio unlock.
The PWA must handle browser timer limits.

The bridge can add native audio, native timer, and local filesystem storage in v1.5.
Cloud sync remains a later feature.

## Alternatives Rejected

### Backend first

A backend first design would make network access a requirement.
It would conflict with the locked zero-backend decision for v1.

### Bridge first

A bridge first design would make paid desktop software a requirement.
It would reduce the value of the free browser version.
