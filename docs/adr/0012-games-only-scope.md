# ADR-0012: Family games only

- **Status:** Accepted
- **Date:** 2026-09-26
- **Supersedes:** The scam-checker scope retained in ADR-0010 and references to it in ADR-0006 and ADR-0008. Other decisions in those records remain in force.

## Context
The games-first pivot left a scam checker in the architecture, roadmap, and older decision records. The product is now a private place for family and group games. A scam checker does not serve that purpose.

## Decision
- Remove the scam checker from the product plan, including text and screenshot flows, AI jobs, and roadmap tasks.
- Keep Chef focused on hosting games, family-safe banter, and optional table talk.
- Keep earlier ADRs as historical records; use this ADR for the current scope.

## Consequences
- The first release has a clearer promise: short games that families play together.
- No scam-checker UI, prompts, screenshot pipeline, or verdict language need to be built.
- The existing no-E2E decision still applies to AI-hosted game content and optional table talk; its older scam-checker rationale is historical.
