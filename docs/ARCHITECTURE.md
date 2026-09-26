# Kitchen Table — Architecture (v1)

> Status: **Agreed plan, not yet built.** Changes to anything below go through a new
> [decision record](adr/README.md).

## 1. Product scope

A private, invite-only family chat app for **web, iOS and Android**, with an AI assistant
(Claude on Amazon Bedrock) built into conversations, and a **scam checker** for parents.

### In v1
- Passwordless sign-in; create a family; invite members with a code/link
- Family group chats and 1:1 direct messages, realtime, with photos
- Private chat with the AI, and `@ai` mentions inside any chat
- "Is this a scam?" — paste text or upload a screenshot, get a plain-language verdict
- Parent-friendly UI: large text mode, voice input, read-aloud
- Push notifications
- English only (built i18n-ready — see [ADR-0009](adr/0009-english-first-i18n-ready.md))

### Explicit non-goals for v1
- Video/voice calls
- Public profiles, feeds, or anything outside the family
- End-to-end encryption — it would prevent the AI from reading the chat
  (see [ADR-0008](adr/0008-no-e2e-encryption-v1.md))
- Public App Store / Play Store listing (internal test builds only — see [ADR-0002](adr/0002-expo-universal-app.md))

## 2. Key decisions

| Area | Choice | Why | ADR |
|---|---|---|---|
| Language | **TypeScript** (strict) everywhere | One language for app, backend and infra | [0001](adr/0001-typescript-monorepo.md) |
| Repo | **pnpm workspaces + Turborepo** monorepo | Shared types, one CI, fast builds | [0001](adr/0001-typescript-monorepo.md) |
| App | **Expo (React Native) + Expo Router** → web, iOS, Android | One codebase; native push that parents can rely on | [0002](adr/0002-expo-universal-app.md) |
| Backend | **API Gateway (HTTP + WebSocket) + Lambda**, **Hono** router | ~$0 idle cost, no servers to patch | [0003](adr/0003-serverless-backend.md) |
| Database | **DynamoDB**, single-table | Chat access patterns are known and key-based | [0004](adr/0004-dynamodb-single-table.md) |
| Auth | **Cognito passwordless** (email OTP + passkeys) | No passwords for parents to forget | [0005](adr/0005-cognito-passwordless.md) |
| AI | **Bedrock Converse API** (streaming), via **SQS** worker | Async, retryable, streamed to clients | [0006](adr/0006-bedrock-ai-via-queue.md) |
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
   │  HTTPS (REST)            │  WebSocket (live)          │ media upload
   ▼                          ▼                            ▼
 API Gateway HTTP API    API Gateway WebSocket API    S3 (presigned PUT)
   │  Cognito JWT auth        │  JWT checked on $connect   │
   ▼                          ▼                        CloudFront (signed GET)
 Lambda: api (Hono)      Lambda: realtime
   │                          │
   ├────────► DynamoDB ◄──────┤
   │
   ├──► SQS: ai-jobs  ──► Lambda: ai-worker ──► Amazon Bedrock (ConverseStream)
   │                          │
   │                          └──► streams chunks to clients via WebSocket
   │
   └──► SQS: notify   ──► Lambda: notifier  ──► Expo Push ──► APNs / FCM
```

Every SQS queue has a **dead-letter queue** and a CloudWatch alarm on it.

## 4. Core flows

### 4.1 Sending a message
1. Client sends `POST /conversations/{id}/messages` (or the WebSocket `sendMessage` action).
2. `api` validates the body with Zod and **checks the caller is a member of the family and
   the conversation**. This authorization check runs on every request — no exceptions.
3. The message is written to DynamoDB with a ULID id (sortable by time).
4. Fan-out: the message is pushed to every online member's WebSocket connection.
   Members with no live connection get a job on `notify` → push notification.
5. If the message mentions `@ai`, or the conversation is an AI chat, a job goes on `ai-jobs`.

### 4.2 AI reply
1. `ai-worker` loads the last *N* messages of the conversation and the asking user's profile
   (name, language).
2. Checks the user's daily AI budget; if exceeded, replies with a friendly limit message.
3. Calls Bedrock `ConverseStream` with a system prompt, Bedrock Guardrails attached.
4. Streams text chunks to the conversation's live connections as `ai.delta` events.
5. Saves the final message, records token usage, sends `ai.done`.

### 4.3 Scam check
1. User pastes text or uploads a screenshot (S3 presigned upload).
2. `api` enqueues an `ai-jobs` item of type `scam_check`.
3. `ai-worker` calls the stronger model with a structured-output prompt returning:
   `verdict` (`likely_scam` | `suspicious` | `looks_safe`), `reasons[]`, `what_to_do[]`.
4. Result is shown as a big, clear card. (Later: optional alert to the family.)

### 4.4 Sign-in and joining a family
1. Email → one-time code (or passkey on returning devices) via Cognito.
2. First sign-in creates a `USER` profile.
3. A member creates a family, or redeems an invite code (expires after 7 days, single use).

## 5. Code structure

Clean / hexagonal architecture: business rules live in `core` and know nothing about AWS.

```
apps/
  mobile/          Expo app — screens and UI only
  api/             Lambda HTTP handlers (thin: validate → call core → respond)
  realtime/        Lambda WebSocket handlers ($connect, $disconnect, sendMessage)
  workers/         ai-worker, notifier
packages/
  shared/          Zod schemas, API types, event types (used by app AND backend)
  core/            Domain: families, members, conversations, messages, permissions
                   Defines ports (interfaces): MessageRepo, AiClient, Notifier, ...
  adapters/        Implementations of the ports: DynamoDB, Bedrock, S3, Expo Push
  config/          Shared tsconfig, ESLint, Prettier configs
infra/             AWS CDK app
  stacks/          AuthStack, DataStack, ApiStack, RealtimeStack, AiStack, WebStack
docs/
  ARCHITECTURE.md  this file
  ROADMAP.md       phases and checklists
  adr/             decision records
```

**Rules**
- `core` imports only `shared`. Never `@aws-sdk/*`.
- Handlers in `apps/*` are thin; logic lives in `core`.
- Every external input is parsed with a Zod schema from `shared` before use.

## 6. Data model (DynamoDB, single table)

Table: `kitchen-table-<stage>`, keys `PK` / `SK`, TTL attribute `expiresAt`.

| Entity / access pattern | PK | SK | Notes |
|---|---|---|---|
| User profile | `USER#<userId>` | `PROFILE` | name, email, `language` (default `en`), text size |
| Families a user belongs to | `USER#<userId>` | `FAMILY#<familyId>` | role, joinedAt |
| Family details | `FAMILY#<familyId>` | `META` | name, createdBy |
| Members of a family | `FAMILY#<familyId>` | `MEMBER#<userId>` | role: `owner` / `member` |
| Conversations in a family | `FAMILY#<familyId>` | `CONV#<convId>` | type: `group` / `dm` / `ai` |
| Conversation members | `CONV#<convId>` | `MEMBER#<userId>` | lastReadAt |
| Messages (newest first, paged) | `CONV#<convId>` | `MSG#<ulid>` | author (`USER#…` or `AI`), text, media |
| Live connections of a user | `USER#<userId>` | `CONN#<connectionId>` | TTL 2h |
| Connection → user lookup | GSI1 `CONN#<connectionId>` | `CONN` | for `$disconnect` cleanup |
| Invite codes | `INVITE#<code>` | `INVITE` | familyId, TTL 7 days |
| Push tokens | `USER#<userId>` | `PUSH#<token>` | platform |
| AI usage | `USER#<userId>` | `USAGE#<yyyy-mm-dd>` | input/output tokens, TTL 90 days |

## 7. AI design

- **Models:** a fast, low-cost Claude model (Haiku class) for chat; a stronger Claude model
  (Sonnet class) for scam checks and hard questions. Exact Bedrock model/inference-profile
  IDs are pinned in config during Phase 2 — never hard-coded in handlers.
- **Guardrails:** Amazon Bedrock Guardrails on every call (harmful content, PII filters).
- **Context:** last *N* messages of the conversation + user name/language. No long-term
  "memory" in v1.
- **Tone:** system prompt tuned for parents: short answers, plain words, no jargon, step-by-step.
- **Cost control:** per-user daily token budget, AWS Budgets alarm, prompt caching where it helps.
- **Privacy:** Bedrock does not use prompts/responses to train models. Message text is never logged.

## 8. Security & privacy

- Bedrock, DynamoDB and S3 are only reached from Lambda via IAM roles —
  **no AWS credentials ever ship in the app**.
- Least-privilege IAM: each Lambda gets only the table actions / queues it needs.
- Authorization on every request: family + conversation membership.
- Media is private: presigned uploads, CloudFront signed URLs with short expiry.
- Encryption in transit (TLS) and at rest (AWS-managed keys).
- Logs contain IDs and metrics, never message bodies.
- Account hygiene: root user MFA, no daily root use, IAM Identity Center logins.

## 9. Environments & deployment

- **One AWS account**, two CDK stages: `dev` and `prod` ([ADR-0007](adr/0007-single-aws-account.md)).
- Every resource is prefixed with its stage (`kitchen-table-dev-…`, `kitchen-table-prod-…`).
- Prod guardrails: DynamoDB deletion protection + point-in-time recovery,
  S3 `RETAIN` removal policy + versioning, stack termination protection.
- GitHub Actions → AWS via OIDC role. `main` deploys `dev` automatically; `prod` deploys on a tagged release.
- Mobile: EAS Build → Google Play internal testing (and TestFlight if anyone uses an iPhone).

## 10. Observability

- AWS Lambda Powertools (TypeScript): structured logs, metrics, tracing.
- CloudWatch alarms: Lambda errors, DLQ depth, Bedrock throttling, budget.
- One small CloudWatch dashboard per stage.

## 11. Cost estimate (family scale, ~10 users)

Rough estimate, not a quote:

| Item | Est. / month |
|---|---|
| Lambda, API Gateway, DynamoDB, SQS, S3, CloudFront | $0–5 (mostly free tier) |
| Cognito (email OTP) | $0 at this scale |
| Bedrock (Claude) | $2–15, depends on usage |
| **Total AWS** | **~$5–20** |
| Google Play developer | $25 one-time |
| Apple Developer (only if needed) | $99 / year |
