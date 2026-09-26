# Kitchen Table — Architecture (v1)

> Status: **Agreed plan, not yet built.** Changes to anything below go through a new
> [decision record](adr/README.md).

## 1. Product scope

**Family game night, a few minutes a day.** A private, invite-only place where a family plays
small games together, with an AI host (Claude on Amazon Bedrock), for **web, iOS and Android**.
Chat exists as *table talk* around the games, not as the main feature
(see [ADR-0010](adr/0010-games-first.md)).

### Design principles
- **1–5 minutes.** Everything fits in a coffee break; parents never need a tutorial.
- **Async and turn-based.** Everyone plays when they can; no need to be online together.
- **About each other.** The best content is the family itself.
- **Cooperative first.** Streaks and milestones belong to the whole family; leaderboards are friendly.

### Games and features (in build order)
| # | Feature | Type |
|---|---|---|
| 1 | **Daily Word Puzzle** — 5 letters, 6 guesses, same word for everyone each day, shareable result | Real game (logic) |
| 2 | Family accounts + **daily family leaderboard** | Platform |
| 3 | **Daily Question** (answers revealed after you answer) + **AI host "Chef"** | Party game |
| 4 | **Arcade mini-game** (e.g. "Catch the Chapati") with a family high-score board, built with **Phaser** | Real game (engine) |
| 5 | **Trivia Night**, **Who Knows Mom Best?**, table-talk chat | Party games + social features |

### Later
Two Truths & a Lie · Caption This · collaborative AI-illustrated story · AI cartoon family photos ·
**Family Recipe & Story Book** (voice → keepsake book) · reminders · more languages · public store launch.

### Explicit non-goals for v1
- Live real-time multiplayer (everything is turn-based / async)
- Video/voice calls
- Public profiles, feeds, or anything outside the family
- End-to-end encryption (would block the AI host — see [ADR-0008](adr/0008-no-e2e-encryption-v1.md))
- Public App Store / Play Store listing (see [ADR-0002](adr/0002-expo-universal-app.md))
- Languages other than English (built i18n-ready — see [ADR-0009](adr/0009-english-first-i18n-ready.md))

## 2. Key decisions

| Area | Choice | Why | ADR |
|---|---|---|---|
| Language | **TypeScript** (strict) everywhere | One language for app, games, backend and infra | [0001](adr/0001-typescript-monorepo.md) |
| Repo | **pnpm workspaces + Turborepo** monorepo | Shared types, one CI, fast builds | [0001](adr/0001-typescript-monorepo.md) |
| App | **Expo (React Native) + Expo Router** → web, iOS, Android | One codebase; native push for parents | [0002](adr/0002-expo-universal-app.md) |
| Game logic | **`packages/games`: pure TypeScript**, no UI, no AWS | Same rules run in app and on server (anti-cheat), easy to test | [0011](adr/0011-game-logic-and-phaser.md) |
| Arcade engine | **Phaser 3** (web canvas; WebView on native) | Best-documented TS 2D engine; great for learning | [0011](adr/0011-game-logic-and-phaser.md) |
| Backend | **API Gateway (HTTP + WebSocket) + Lambda**, **Hono** router | ~$0 idle cost, no servers to patch | [0003](adr/0003-serverless-backend.md) |
| Database | **DynamoDB**, single-table | Access patterns are known and key-based | [0004](adr/0004-dynamodb-single-table.md) |
| Auth | **Cognito passwordless** (email OTP + passkeys) | No passwords for parents to forget | [0005](adr/0005-cognito-passwordless.md) |
| AI | **Bedrock Converse API** (streaming), via **SQS** worker | Async, retryable, streamed to clients | [0006](adr/0006-bedrock-ai-via-queue.md) |
| Schedules | **EventBridge Scheduler** | Daily question / puzzle rollover, weekly trivia & recap | [0010](adr/0010-games-first.md) |
| Files | **S3** presigned uploads + **CloudFront** signed URLs | Private family media | — |
| Push | **Expo Push Service** | Free, one API for iOS + Android | — |
| IaC | **AWS CDK (TypeScript)** | AWS-native, same language | [0001](adr/0001-typescript-monorepo.md) |
| Validation | **Zod** schemas shared by app and backend | One source of truth for data shapes | — |
| Testing | **Vitest** (unit/integration), DynamoDB Local | Fast, TS-native | — |
| CI/CD | **GitHub Actions + AWS OIDC**; **EAS** for mobile builds | No long-lived AWS keys | — |
| AWS accounts | **One account**, `dev` and `prod` stages with guardrails | Simple to start; revisit later | [0007](adr/0007-single-aws-account.md) |

## 3. System overview

```
 Expo app (web / iOS / Android)
   ├─ screens (React Native)
   ├─ packages/games (pure TS rules)       ├─ Phaser arcade (canvas / WebView)
   │
   │  HTTPS (REST)            │  WebSocket (live)          │ media upload
   ▼                          ▼                            ▼
 API Gateway HTTP API    API Gateway WebSocket API    S3 (presigned PUT)
   │  Cognito JWT auth        │  JWT checked on $connect   │
   ▼                          ▼                        CloudFront (signed GET)
 Lambda: api (Hono)      Lambda: realtime
   │  uses packages/games     │
   │  to verify results       │
   ├────────► DynamoDB ◄──────┤
   │
   ├──► SQS: ai-jobs  ──► Lambda: ai-worker ──► Amazon Bedrock (ConverseStream)
   │                          └──► streams "Chef" messages to clients via WebSocket
   │
   └──► SQS: notify   ──► Lambda: notifier  ──► Expo Push ──► APNs / FCM

 EventBridge Scheduler ──► Lambda: scheduler (daily drop, weekly trivia, weekly recap)
```

Every SQS queue has a **dead-letter queue** and a CloudWatch alarm on it.

## 4. Core flows

### 4.1 Daily Word Puzzle
- **Step 1 (no backend):** the puzzle number is derived from the date; the word is picked
  from a bundled word list by that number. Everyone gets the same word. Progress is saved on
  the device. The share text looks like `Kitchen Table #12 4/6` + emoji grid.
- **Step 2+ (with backend):** the word list moves server-side. The client sends its guesses;
  `api` **replays them with `packages/games`** against the real answer and records the verified
  result. The family leaderboard is built only from verified results.

### 4.2 Arcade high score
1. The Phaser game runs locally and produces a score plus a small run summary
   (duration, events).
2. `api` runs **plausibility checks** (max points/second, duration limits) before storing
   the family best score. Perfect anti-cheat is out of scope for a family app.

### 4.3 Daily Question
1. EventBridge Scheduler triggers the `scheduler` Lambda once a day per family.
2. A question is picked from a curated bank, or generated by the AI host with the family's
   past topics as context, and stored for that date.
3. Members answer. **Answers are only returned to a member after they have answered.**
4. Chef posts a playful summary once everyone has answered (via `ai-jobs`).

### 4.4 AI host "Chef"
1. Game events (daily summary, trivia rounds, winners, weekly recap) enqueue jobs on `ai-jobs`.
2. `ai-worker` builds the prompt: Chef's persona, the game event, relevant family context.
3. Calls Bedrock `ConverseStream` with Guardrails; streams to clients; saves the message.
4. Per-family daily AI budget enforced by the worker.

### 4.5 Table talk (chat)
1. Each game/day has a table-talk thread for reactions and banter.
2. `api` validates with Zod and **checks family membership on every request**.
3. Messages are fanned out over WebSocket; offline members get a push via `notify`.
4. `@chef` mentions enqueue an AI reply.

### 4.6 Sign-in and joining a family
1. Email → one-time code (or passkey on returning devices) via Cognito.
2. First sign-in creates a `USER` profile.
3. A member creates a family, or redeems an invite code (expires after 7 days, single use).

## 5. Code structure

Clean / hexagonal architecture: business and game rules know nothing about UI or AWS.

```
apps/
  mobile/          Expo app — screens and UI only
  arcade/          Phaser games (built for web; embedded via WebView on native)
  api/             Lambda HTTP handlers (thin: validate → call core → respond)
  realtime/        Lambda WebSocket handlers ($connect, $disconnect, sendMessage)
  workers/         ai-worker, notifier, scheduler
packages/
  games/           Pure TS game rules: word puzzle, scoring, streaks, trivia rounds
  shared/          Zod schemas, API types, event types (used by app AND backend)
  core/            Domain: families, members, games, questions, messages, permissions
                   Defines ports (interfaces): GameRepo, AiClient, Notifier, ...
  adapters/        Implementations of the ports: DynamoDB, Bedrock, S3, Expo Push
  config/          Shared tsconfig, ESLint, Prettier configs
infra/             AWS CDK app
  stacks/          AuthStack, DataStack, ApiStack, RealtimeStack, AiStack, SchedulerStack, WebStack
docs/
  ARCHITECTURE.md  this file
  ROADMAP.md       build steps and checklists
  adr/             decision records
```

**Rules**
- `packages/games` imports nothing but TypeScript itself: no React, no Phaser, no AWS.
  Functions are deterministic (randomness comes from an injected seed) so they are easy to test.
- `core` imports only `shared` and `games`. Never `@aws-sdk/*`.
- Handlers in `apps/*` are thin; logic lives in `core` / `games`.
- Every external input is parsed with a Zod schema from `shared` before use.

## 6. Data model (DynamoDB, single table)

Table: `kitchen-table-<stage>`, keys `PK` / `SK`, TTL attribute `expiresAt`.

| Entity / access pattern | PK | SK | Notes |
|---|---|---|---|
| User profile | `USER#<userId>` | `PROFILE` | name, email, `language` (default `en`), text size |
| Families a user belongs to | `USER#<userId>` | `FAMILY#<familyId>` | role, joinedAt |
| Family details | `FAMILY#<familyId>` | `META` | name, timezone, createdBy |
| Members of a family | `FAMILY#<familyId>` | `MEMBER#<userId>` | role: `owner` / `member` |
| Family streak | `FAMILY#<familyId>` | `STREAK` | current, best, lastFullDay |
| Puzzle results for a day (leaderboard) | `FAMILY#<familyId>` | `PUZZLE#<yyyy-mm-dd>#<userId>` | guesses, solved, verified |
| Arcade best scores | `FAMILY#<familyId>` | `SCORE#<gameKey>#<userId>` | best, achievedAt |
| Daily question | `FAMILY#<familyId>` | `DQ#<yyyy-mm-dd>` | question text, source (`bank` / `ai`) |
| Daily question answers | `DQ#<familyId>#<yyyy-mm-dd>` | `ANSWER#<userId>` | text; read only after caller answered |
| Game session (trivia etc.) | `GAME#<gameId>` | `META` / `ROUND#<n>` / `PLAYER#<userId>` | state, scores |
| Conversations (table talk) | `FAMILY#<familyId>` | `CONV#<convId>` | linked game/day, or DM |
| Messages (newest first, paged) | `CONV#<convId>` | `MSG#<ulid>` | author (`USER#…` or `CHEF`), text, media |
| Live connections of a user | `USER#<userId>` | `CONN#<connectionId>` | TTL 2h |
| Connection → user lookup | GSI1 `CONN#<connectionId>` | `CONN` | for `$disconnect` cleanup |
| Invite codes | `INVITE#<code>` | `INVITE` | familyId, TTL 7 days |
| Push tokens | `USER#<userId>` | `PUSH#<token>` | platform |
| AI usage | `FAMILY#<familyId>` | `USAGE#<yyyy-mm-dd>` | input/output tokens, TTL 90 days |

## 7. AI design (Chef)

- **Persona:** "Chef", the warm, slightly cheeky host of the Kitchen Table. Short messages,
  plain words, gentle teasing, never mean. Persona lives in one versioned system prompt.
- **Jobs:** daily question generation, daily-question summaries, trivia question generation,
  winner announcements, weekly recap, `@chef` replies.
- **Models:** a fast, low-cost Claude model (Haiku class) for host chatter; a stronger Claude
  model (Sonnet class) for trivia generation. Exact Bedrock model/inference-profile
  IDs are pinned in config when AI work starts — never hard-coded in handlers.
- **Guardrails:** Amazon Bedrock Guardrails on every call.
- **Trivia quality:** AI-generated questions are validated with structured output
  (question, 4 options, answer index, source hint) and can be regenerated if malformed.
- **Cost control:** per-family daily token budget, AWS Budgets alarm, prompt caching for the persona prompt.
- **Privacy:** Bedrock does not use prompts/responses to train models. Message text is never logged.

## 8. Security & privacy

- Bedrock, DynamoDB and S3 are only reached from Lambda via IAM roles —
  **no AWS credentials ever ship in the app**.
- Least-privilege IAM: each Lambda gets only the table actions / queues it needs.
- Authorization on every request: family (and conversation/game) membership.
- Game results are verified or plausibility-checked server-side before hitting leaderboards.
- Media is private: presigned uploads, CloudFront signed URLs with short expiry.
- Encryption in transit (TLS) and at rest (AWS-managed keys).
- Logs contain IDs and metrics, never message bodies or answers.
- Account hygiene: root user MFA, no daily root use, IAM Identity Center logins.

## 9. Environments & deployment

- **Step 1 has no AWS at all** — the web build can be hosted anywhere static (or just run locally).
- From step 2: **one AWS account**, two CDK stages: `dev` and `prod` ([ADR-0007](adr/0007-single-aws-account.md)).
- Every resource is prefixed with its stage (`kitchen-table-dev-…`, `kitchen-table-prod-…`).
- Prod guardrails: DynamoDB deletion protection + point-in-time recovery,
  S3 `RETAIN` removal policy + versioning, stack termination protection.
- GitHub Actions → AWS via OIDC role. `main` deploys `dev` automatically; `prod` deploys on a tagged release.
- Mobile: EAS Build → Google Play internal testing (and TestFlight if anyone uses an iPhone).

## 10. Observability

- AWS Lambda Powertools (TypeScript): structured logs, metrics, tracing.
- CloudWatch alarms: Lambda errors, DLQ depth, Bedrock throttling, budget.
- One small CloudWatch dashboard per stage.
- Product metrics (counts only): daily players, streak length, games played.

## 11. Cost estimate (family scale, ~10 users)

Rough estimate, not a quote:

| Item | Est. / month |
|---|---|
| Step 1 (static web, no AWS) | $0 |
| Lambda, API Gateway, DynamoDB, SQS, S3, CloudFront, Scheduler | $0–5 (mostly free tier) |
| Cognito (email OTP) | $0 at this scale |
| Bedrock (Chef + trivia) | $2–15, depends on usage |
| **Total AWS** | **~$5–20** |
| Google Play developer | $25 one-time |
| Apple Developer (only if needed) | $99 / year |
