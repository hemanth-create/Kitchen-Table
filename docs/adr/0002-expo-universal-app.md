# ADR-0002: Expo universal app; web + internal test builds first

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
We need web, iOS and Android. Parents need reliable push notifications and a real app icon.
Web-only (PWA) push on iOS requires "Add to Home Screen", which parents won't do.
A public store launch adds review and listing overhead we don't need yet.

## Decision
- One **Expo (React Native) + Expo Router** codebase targeting web, iOS and Android.
- Ship the **web app** plus **internal test builds** (Google Play internal testing;
  TestFlight only if a family member uses an iPhone). No public store listing in v1.

## Consequences
- Native push works from day one.
- React Native on web is slightly less flexible than a pure web framework — acceptable for a chat UI.
- Going public later is a store listing + review, not a rewrite.
- Costs: $25 one-time (Google); $99/year (Apple) only if needed.
