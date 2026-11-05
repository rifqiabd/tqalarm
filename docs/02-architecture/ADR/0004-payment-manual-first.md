# ADR 0004: Manual Payment First

- **Status:** Accepted
- **Date:** 2026-11-05

## Context

TQAlarm has a free browser version and a paid bridge version.
The product must validate the paid path without building payment software in v1.

The product decision requires manual payment through WhatsApp first.
Automatic payment services would add scope and external dependencies.

## Decision

TQAlarm uses manual payment through WhatsApp first.
The v1 product does not build an automatic payment system.
The v1 product does not integrate Midtrans.
The v1 product does not integrate Xendit.

Payment automation is outside the API contract.
Payment automation is outside the PWA v1 scope.
Payment automation is outside the bridge v1.5 architecture defined here.

## Consequences

The team can support payment through a manual WhatsApp process.
The product does not need payment API credentials in v1.
The product does not need payment webhooks in v1.
The product must keep payment implementation details outside these architecture contracts.

Later payment automation requires a new decision.
Later payment automation requires a scope review.

## Alternatives Rejected

### Automatic payment in v1

This option would add payment system scope.
It would conflict with the locked manual-first decision.

### Midtrans or Xendit in v1

These integrations would add external payment dependencies.
They would conflict with the product backlog and v1 limits.
