# ADR-0001: TypeScript monorepo

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
The app, backend and infrastructure all need to be written, by a small team that wants to
learn TypeScript. Types shared between app and backend prevent a whole class of bugs.

## Decision
- TypeScript in **strict** mode for everything: app, Lambdas, infrastructure.
- One repo using **pnpm workspaces** and **Turborepo** (`apps/*`, `packages/*`, `infra/`).
- Infrastructure as code with **AWS CDK** in TypeScript.
- **Zod** schemas in `packages/shared` are the single source of truth for API shapes.

## Consequences
- One language, one toolchain, one CI pipeline.
- Changing an API type breaks the build on both sides immediately — intended.
- Monorepo tooling has a small learning curve (workspaces, task caching).
