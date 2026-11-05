# TQAlarm Testing

## Test scope

Version 1 is browser-only, offline-first, and zero-backend. Its acceptance checks cover schedules, bells, browser audio, local storage, data transfer, onboarding, and the basic noise generator. Bridge behavior is not a v1 acceptance requirement; it is identified separately as a v1.5 boundary.

Use Vitest for focused logic and data tests, Playwright for browser flows, and `svelte-check` for Svelte and TypeScript diagnostics. Do not add unapproved coverage thresholds, browser matrices, ports, or CI commands.

## Required checks

### 1. Browser-only and offline behavior

- Open the PWA without a bridge and confirm the dashboard works in the browser.
- Confirm v1 does not require a backend or cloud connection for its core flows.
- Confirm the application can create, edit, activate, and deactivate schedules while offline.
- Confirm the supported local data remains available after closing and reopening the browser.
- Confirm `/` is the dashboard, `/settings` is settings, and `/onboarding` is onboarding. Do not treat `/display` as a v1 feature; `/kiosk` is prepared for the bridge stage.
- Confirm a damaged custom sound can fall back to the default tone.

### 2. Audio unlock

- Open the application in a browser state where audio has not been unlocked.
- Confirm the audio-unlock banner is visible and its status is understandable.
- Use the explicit unlock action and confirm the AudioContext is resumed.
- Use **Test Bell** after unlock and confirm the selected test sound is audible.
- Reload or reopen the application and verify the stored audio-unlock state is handled without claiming native or bridge audio behavior.
- Confirm the browser flow does not assume that autoplay is available before user interaction.

### 3. Timer and leader election

- Create an active bell and verify the Web Worker timer evaluates it at the intended time.
- Open multiple tabs for the same local application and confirm leader election allows only one tab to emit a bell.
- Change or deactivate a schedule and confirm the timer does not continue to emit the old active bell.
- Exercise a date override for `holiday`, `exam`, `event`, and `custom` and confirm the selected override controls the applicable bell behavior.
- Confirm a temporary tab or worker interruption does not create duplicate bells when leadership is re-established.
- Record failures as browser timer or leader-election issues; do not attribute them to the v1.5 bridge.

### 4. Import and export

- Export a supported configuration to JSON.
- Import the exported JSON into the browser and confirm supported schedules, bells, overrides, sounds, settings, and presets are preserved as applicable.
- Confirm a round trip does not silently lose supported data.
- Import a supported Excel schedule through SheetJS and verify the resulting schedule and bell data.
- Verify invalid or unsupported input is rejected or reported without corrupting existing local data.
- Confirm import/export remains local in v1 and does not require Supabase or automated cloud backup.

### 5. Onboarding

- Start with a new local browser profile or otherwise empty application state.
- Confirm onboarding has exactly the intended three-step flow.
- Confirm the flow explains the relevant setup state and provides the required **Test Bell** button.
- Complete onboarding and verify that the operator can hear the test bell without technical assistance.
- Confirm the resulting application state is available on the dashboard and persists locally.
- Confirm Indonesian text is complete for the v1 onboarding flow; English is a ready structure, not a claim of complete English translation.

### 6. v1 boundary

- Confirm v1 does not require or include bridge desktop execution, native audio, native timer, SQLite as source of truth, filesystem MP3 storage, system tray, auto-start, watchdog, offline license validation, or auto-update.
- Confirm v1 does not include display board UI, academic calendar integration, complete English localization, school branding UI, automated payments, Google Drive backup, social sharing, or cloud sync.
- Confirm payment remains a manual WhatsApp process and no payment system is exercised by v1 tests.

## v1.5 bridge boundary checks

These checks are planning boundaries for the later bridge stage, not v1 release gates:

- The PWA can connect to a bridge through authenticated `localhost` WebSocket transport.
- The bridge becomes source of truth when connected, while the browser uses IndexedDB as fallback when it is not connected.
- The bridge can receive approved configuration data and provide native audio and native timer behavior.
- Bridge settings are stored in SQLite, and MP3 files are stored in the local filesystem.
- Bells continue when the browser is closed, subject to the approved v1.5 implementation.
- Any kiosk, system tray, auto-start, watchdog, offline license validation, and auto-update checks are assessed only against the v1.5 specification.

The transport port, endpoint, handshake, token format, message schema, reconnection policy, and bridge test fixtures are not defined by the current product documents. Do not invent them in tests before an approved contract exists.

## Test evidence

For each failed check, record the scenario, browser state, input data, expected result, observed result, and whether the issue belongs to v1 or the v1.5 bridge boundary. Keep v1 results separate from bridge feedback so that bridge expectations do not block the browser-only release.
