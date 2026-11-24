# ADR-0004: PWA mode (installed vs Safari tab vs website)

- Status: accepted (installed PWA); confirmed by T1-T6 and the 8-day storage test
- Date: 2026-11-24
- Deciders: team

## Context

Sessions run offline-first on iPadOS and Android tablets. Unsent records live in IndexedDB. Safari deletes script-writable storage, including IndexedDB, after 7 days of Safari use without interaction with the site. Web apps added to the Home Screen have their own days-of-use counter (documented by WebKit in 2020).

## Options

1. Installed (Home Screen) PWA.
2. PWA used in a Safari tab.
3. Plain website.

## Decision

The installed PWA is required in production. The service worker never calls `skipWaiting()` or `clients.claim()`, so a new version activates at the next start, never mid-session. The start screen shows the app version and the number of unsent records.

## Consequences

The nurse installs the app once per tablet (setup link or QR). Browser-tab use is not supported for production.

## Evidence

On the newest iPadOS and one Android tablet, per mode:

- T1 start-up time (10 cold, 10 warm).
- T2 offline: 5 sessions offline, restart, reconnect; records sent vs created, duplicates.
- T3 camera and microphone prompts over 3 launches.
- T4 update behaviour during and between sessions.
- T5 screen wake lock in a 5-minute session.
- T6 setup effort with a stopwatch.
- 7-day storage test (protocol below): records in the outbox, app closed for 8 days, Safari used daily on other sites; records still present yes/no. One device, one run.

Protocol for the 7-day test: write the device model, iPadOS version, start date, number of records and the exact steps here before starting.
