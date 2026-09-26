# Kitchen Table — Architecture (v1)

> Status: **Agreed plan, not yet built.** Changes to anything below go through a new
> [decision record](adr/README.md).

## 1. Product scope

**Short games that families and friends play together.** A private place where a group — a
**Table** — plays small, turn-based games, with an optional AI host "Chef" (Claude on Amazon
Bedrock). Web first, then iOS and Android. Players are adults.
See [ADR-0010](adr/0010-games-first.md), [ADR-0012](adr/0012-games-only-scope.md),
[ADR-0013](adr/0013-shared-game-first.md).

The product test: **do people come back to play with each other?** Not: do they use an AI feature.

### Design principles
- **1–5 minutes.** Everything fits in a coffee break; no tutorial needed.
- **Async and turn-based.** Every round has a deadline; nobody needs to be online at the same time.
- **About each other.** The best content is the group itself.
- **Cooperative first.** Milestones belong to the whole Table and never name who missed.
- **Works without AI.** Rules, scores, visibility and deadlines are plain code; Chef only adds flavour.

### Tables
- A **Table** is a private group: a family, a friend group, etc. A user can sit at several Tables.
- Each Table has a name, a timezone, and 3–12 players.

### Games (in build order)
| # | Game | Type | Rules |
|---|---|---|---|
| 1 | **Who Knows Us Best?** — one player picks an answer, others guess, reveal together | Shared, async | [rules](games/who-knows-us-best.md) |
| 2 | **Daily Question** — everyone answers one prompt; answers revealed at close | Shared, async | to write before Step 4 |
| 3 | **Story Relay** — Chef opens a scenario; players add one line each or pick a twist | Shared, async | to write before Step 5 |
| 4 | **Quick polls / This or That** — 30-second fillers between games | Shared, async | to write before Step 5 |

### Later
Caption This (needs private media + consent) · Trivia Night (reviewed question bank only) ·
Daily Word Puzzle and Phaser arcade games (optional learning track, [ADR-0013](adr/0013-shared-game-first.md)) ·
Family Recipe & Story Book · more languages · public store launch.

### Explicit non-goals for v1
- Live real-time sessions and WebSockets ([ADR-0014](adr/0014-async-only-v1.md))
- Children as players
- Chat as a headline feature; video/voice calls
- Public profiles, feeds, or anything outside a Table
- End-to-end encryption ([ADR-0008](adr/0008-no-e2e-encryption-v1.md))
- AI-generated **scored** content (e.g. trivia answers) without human review
- Public App Store / Play Store listing ([ADR-0002](adr/0002-expo-universal-app.md))
- Languages other than English ([ADR-0009](adr/0009-english-first-i18n-ready.md))

## 2. Key decisions

| Area | Choice | Why | ADR |
|---|---|---|---|
| Language | **TypeScript** (strict) everywhere | One language for app, games, backend and infra | [0001](adr/0001-typescript-monorepo.md) |
| Repo | **pnpm workspaces + Turborepo** monorepo | Shared types, one CI, fast builds | [0001](adr/0001-typescript-monorepo.md) |
| App | **Expo (React Native) + Expo Router**, web first | One codebase for web, iOS, Android | [0002](adr/0002-expo-universal-app.md) |
| Game logic | **`packages/games`: pure, deterministic TypeScript** | Same rules on client and server; easy to test | [0011](adr/0011-game-logic-and-phaser.md) |
| Backend | **API Gateway HTTP API + Lambda**, **Hono** router | ~$0 idle; the same Hono app runs locally on Node | [0003](adr/0003-serverless-backend.md) |
| Updates | **No WebSockets in v1**: refresh on open/focus, light polling on an open round, push later | All games are async; far simpler | [0014](adr/0014-async-only-v1.md) |
| Database | **DynamoDB**, single-table | Access patterns are known and key-based | [0004](adr/0004-dynamodb-single-table.md) |
| Identity | **Room links** (secret link + device token) for the playtest; **Cognito passwordless** after | Playtest fast, then real accounts | [0015](adr/0015-room-links-before-accounts.md), [0005](adr/0005-cognito-passwordless.md) |
| AI | **Bedrock Converse API** via an **SQS** worker, idempotent, non-streaming | Optional flavour; safe to retry | [0006](adr/0006-bedrock-ai-via-queue.md), [0014](adr/0014-async-only-v1.md) |
| Schedules | **EventBridge Scheduler** (from Step 4) | Daily question drops, "results are in" pushes | [0010](adr/0010-games-first.md) |
| Push | **Expo Push Service** (from Step 4) | Free, one API for iOS + Android | — |
| IaC | **AWS CDK (TypeScript)** | AWS-native, same language | [0001](adr/0001-typescript-monorepo.md) |
| Validation | **Zod** schemas shared by app and backend | One source of truth for data shapes | — |
| Testing | **Vitest** (unit/integration), DynamoDB Local | Fast, TS-native | — |
| CI/CD | **GitHub Actions + AWS OIDC**; **EAS** for mobile builds | No long-lived AWS keys | — |
| AWS accounts | **One account**, `dev` and `prod` stages with guardrails | Simple to start; revisit later | [0007](adr/0007-single-aws-account.md) |

## 3. System overview

```
 Expo app (web first; iOS / Android later)
   ├─ screens (React Native)
   └─ packages/games (pure TS rules — also used by the server)
   │
   │  HTTPS (REST): refresh on open/focus, light polling on an open round
   ▼
 API Gateway HTTP API
   │  room token (playtest) → Cognito JWT (Step 3+)
   ▼
 Lambda: api (Hono) ── uses packages/games for every state change
   │
   ├────────► DynamoDB
   │
   ├──► SQS: ai-jobs ──► Lambda: ai-worker ──► Amazon Bedrock (Converse)
   │                         └─ saves Chef's message once (idempotent)
   │
   └──► SQS: notify  ──► Lambda: notifier ──► Expo Push        (Step 4+)

 EventBridge Scheduler ──► Lambda: scheduler (daily question, reveal pushes)  (Step 4+)
```

Every SQS queue has a **dead-letter queue** and a CloudWatch alarm on it.

## 4. Core flows

### 4.1 Who Knows Us Best? round
Full rules: [games/who-knows-us-best.md](games/who-knows-us-best.md).
1. `POST /tables/{id}/rounds` — `api` checks the caller sits at the Table, asks
   `packages/games` for the next subject and an unused question, and writes the round with a
   UTC deadline (24 h). Only one open round per Table (conditional write).
2. `PUT /rounds/{id}/pick` — subject submits their answer, others their guess. Editable until reveal.
3. `GET /rounds/{id}` — returns the round. **Reveal is computed lazily**: if everyone has
   submitted or the deadline has passed, the round is revealed and picks are included;
   otherwise only "who has submitted" is returned — never the picks.
4. On first reveal, `api` enqueues one Chef job with event ID `ROUND#<id>#reveal`.

### 4.2 Chef reaction (optional)
1. `ai-worker` receives the job. Event ID + a conditional write guarantee at most **one** saved
   Chef message and one usage record per event, even if SQS delivers the job twice.
2. Prompt contains only revealed data: question, subject's answer, known-by score, first names.
3. Calls Bedrock `Converse` with Guardrails and a timeout; saves the message.
4. On error, timeout or budget exceeded, the client shows the plain fallback message instead.
   The game never waits on Chef.

### 4.3 Daily Question (Step 4)
1. EventBridge Scheduler triggers `scheduler` once a day per Table (Table timezone).
2. A prompt is picked from the curated bank (Chef may rephrase it — not required).
3. Players answer; answers are shown to a player after they answer, and to everyone at close
   (end of the Table's day). Same lazy-reveal rule as 4.1.

### 4.4 Joining a Table
- **Playtest (Steps 1–2):** the creator gets a secret room link. Opening it asks for a display
  name and issues a random **device token** stored on the device. The creator's token can
  remove players. ([ADR-0015](adr/0015-room-links-before-accounts.md))
- **Step 3+:** Cognito passwordless sign-in; single-use invite codes redeemed with a
  conditional write (atomic); playtest players can claim their existing seat.
- Every read and write checks the caller sits at that Table — no exceptions.

## 5. Code structure

Clean / hexagonal architecture: game and business rules know nothing about UI or AWS.

```
apps/
  mobile/          Expo app — screens and UI only
  api/             Hono app: runs on Lambda in AWS and on Node locally
  workers/         ai-worker, notifier, scheduler
packages/
  games/           Pure TS game rules: rounds, rotation, reveal, scoring, deadlines
  shared/          Zod schemas, API types (used by app AND backend)
  core/            Domain: tables, players, rounds; permission checks
                   Defines ports (interfaces): TableRepo, RoundRepo, AiClient, Clock, ...
  adapters/        Implementations of the ports: DynamoDB, in-memory (local dev/tests), Bedrock, Expo Push
  config/          Shared tsconfig, ESLint, Prettier configs
infra/             AWS CDK app
docs/
  ARCHITECTURE.md  this file
  ROADMAP.md       build steps and checklists
  games/           one rules page per game + playtest kits
  adr/             decision records
```

**Rules**
- `packages/games` imports nothing but TypeScript itself. Functions are deterministic:
  time comes from an injected clock, randomness from an injected seed.
- `core` imports only `shared` and `games`. Never `@aws-sdk/*`.
- Handlers in `apps/*` are thin; logic lives in `core` / `games`.
- Every external input is parsed with a Zod schema from `shared` before use.
- Every game gets a rules page in `docs/games/` **before** its code is written.

## 6. Data model (DynamoDB, single table)

Table: `kitchen-table-<stage>`, keys `PK` / `SK`, TTL attribute `expiresAt`.

| Entity / access pattern | PK | SK | Notes |
|---|---|---|---|
| Table details | `TABLE#<tableId>` | `META` | name, timezone, createdBy |
| Players at a Table (rotation order) | `TABLE#<tableId>` | `PLAYER#<playerId>` | display name, joinedAt, sittingOut, role |
| Device token → player | `TOKEN#<sha256(token)>` | `TOKEN` | tableId, playerId (playtest only; hash stored, never the token) |
| Rounds at a Table (newest first) | `TABLE#<tableId>` | `ROUND#<ulid>` | game, subject, questionId, deadline, status |
| Open-round lock | `TABLE#<tableId>` | `OPEN_ROUND` | roundId; conditional write → one open round |
| Picks in a round | `ROUND#<roundId>` | `PICK#<playerId>` | option, updatedAt |
| Chef message for an event | `ROUND#<roundId>` | `CHEF#<eventType>` | text; conditional write → idempotent |
| Questions used at a Table | `TABLE#<tableId>` | `USEDQ#<questionId>` | no repeats until bank exhausted |
| Weekly scores | `TABLE#<tableId>` | `WEEK#<yyyy-Www>#<playerId>` | points |
| AI usage | `TABLE#<tableId>` | `USAGE#<yyyy-mm-dd>#<eventId>` | tokens; keyed by event → idempotent |
| **Step 3+:** User profile | `USER#<userId>` | `PROFILE` | name, email, `language` |
| **Step 3+:** Tables a user sits at | `USER#<userId>` | `TABLE#<tableId>` | playerId |
| **Step 3+:** Invite codes | `INVITE#<code>` | `INVITE` | tableId, TTL 7 days, single use |
| **Step 4+:** Push tokens | `USER#<userId>` | `PUSH#<token>` | platform |

The curated question bank ships as versioned JSON in `packages/games` (reviewed by a human).

## 7. AI design (Chef)

- **Role:** optional host. Brief reactions after reveals, round recaps, optional rephrasing of
  curated prompts, Story Relay openers. **Never** decides rules, scores, visibility or deadlines.
- **Persona:** warm, playful, adult-friendly. Teases the *answer*, never a person's score.
  One versioned system prompt.
- **Cost:** one call per game event, shared by the whole Table — never one call per player action.
  Per-Table daily budget; AWS Budgets alarm.
- **Reliability:** stable event IDs, idempotent saves, timeout, plain fallback text. No streaming.
- **Privacy:** Chef only receives revealed content. Players are told when game content is sent to Chef.
  Bedrock does not use prompts/responses to train models. Content is never logged.
- **Models:** a fast, low-cost Claude model (Haiku class). Exact Bedrock model/inference-profile
  IDs are pinned in config when AI work starts.
- **Guardrails:** Amazon Bedrock Guardrails on every call.
- **Accuracy:** no AI-generated scored facts. Trivia (later) uses a human-reviewed bank; Chef may
  only add banter and explanations around it.

## 8. Security & privacy

- **Plain-language privacy note** before any playtest: who can see what, that the app operator
  can technically read game data (no E2E), what is sent to Chef, how to delete your content.
- Room links use unguessable IDs (≥128-bit); device tokens are random and stored hashed.
- Picks are never returned before the reveal — enforced server-side, not just hidden in the UI.
- Players can sit out, skip, and delete their own picks.
- Only Lambda reaches DynamoDB and Bedrock, via least-privilege IAM roles —
  **no AWS credentials ever ship in the app**.
- API Gateway throttling on all routes; per-Table round limit (10/day).
- Encryption in transit (TLS) and at rest (AWS-managed keys). Logs contain IDs and metrics only.
- Account hygiene: root user MFA, no daily root use, IAM Identity Center logins.

## 9. Environments & deployment

- **Local:** the Hono API runs on Node with in-memory adapters — no AWS needed to develop.
- **AWS:** one account, `dev` and `prod` CDK stages ([ADR-0007](adr/0007-single-aws-account.md)),
  resources prefixed with the stage.
- Prod guardrails: DynamoDB deletion protection + point-in-time recovery,
  S3 `RETAIN` + versioning, stack termination protection.
- GitHub Actions → AWS via OIDC. `main` deploys `dev`; `prod` deploys on a tagged release.
- Web app: S3 + CloudFront. Mobile (later): EAS Build → internal testing tracks.

## 10. Observability

- AWS Lambda Powertools (TypeScript): structured logs, metrics, tracing.
- CloudWatch alarms: Lambda errors, DLQ depth, Bedrock errors/throttling, budget.
- **Product metrics (counts only):** rounds started, % of players who pick per round,
  rounds per session ("replay"), Tables active per week.

## 11. Cost estimate (a few Tables, ~10–30 players)

Rough estimate, not a quote:

| Item | Est. / month |
|---|---|
| Step 0 (WhatsApp) and local development | $0 |
| Lambda, API Gateway, DynamoDB, SQS, S3, CloudFront | $0–5 (mostly free tier) |
| Cognito (email OTP) | $0 at this scale |
| Bedrock (Chef: one call per reveal) | $1–5 |
| **Total AWS** | **~$1–10** |
| Google Play developer (later) | $25 one-time |
| Apple Developer (only if needed) | $99 / year |
