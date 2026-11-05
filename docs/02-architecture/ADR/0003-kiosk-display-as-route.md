# ADR 0003: Kiosk and Display as Routes

- **Status:** Accepted
- **Date:** 2026-11-05

## Context

TQAlarm uses a SvelteKit PWA.
The product needs a kiosk route for bridge operation.
The product plans a display route for a later stage.

A separate application would duplicate PWA behavior.
A separate application would increase maintenance work.

## Decision

Kiosk and display are PWA routes.
The kiosk route is `/kiosk`.
The display route is `/display`.
The `/kiosk` route is a v1.5 feature.
The `/display` route is a v2 feature.

The routes are not separate applications.
The routes use the shared PWA data model.
The routes do not change the client-side first architecture.

## Consequences

The PWA keeps one route structure.
The PWA can share schedule and settings data between routes.
The bridge can support kiosk operation through `/kiosk`.
The display route remains outside v1 and v1.5 display UI scope.

The team must not create a separate kiosk application.
The team must not create a separate display application.
The team must preserve `schoolLogo`, `displayMode`, and `announcements` in the data model.

## Alternatives Rejected

### Separate kiosk application

This option would duplicate the PWA route and data behavior.
It would conflict with the locked route decision.

### Separate display application

This option would create a second application for a planned PWA route.
It would add scope outside the approved roadmap.
