# ADR-0009: English first, i18n-ready

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
v1 users read English. Other languages may be wanted later, and retrofitting translation
into a finished UI is expensive.

## Decision
- Ship **English only** in v1.
- All UI strings go through **i18next** (with `expo-localization`) from day one — no hard-coded text.
- Every user profile has a `language` field (default `en`); the AI replies in that language.

## Consequences
- Adding a language later = adding a translation file + testing, not rewriting screens.
- Slight overhead per string during development.
