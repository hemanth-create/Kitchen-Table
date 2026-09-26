# Kitchen Table — Roadmap

Each phase ends with something working you can show your family.

## Phase 0 — Foundations (week 1)

- [ ] AWS: root MFA on, IAM Identity Center user for daily work
- [ ] AWS: Budgets alarm (e.g. $25/month) with email alert
- [ ] AWS: enable Bedrock model access for the chosen Claude models in your region
- [ ] Repo: pnpm + Turborepo monorepo skeleton, strict `tsconfig`, ESLint, Prettier
- [ ] Repo: `packages/shared`, `packages/core`, `packages/adapters`, `infra/` created
- [ ] CI: GitHub Actions — lint, typecheck, test on every PR
- [ ] CI: GitHub → AWS OIDC role; CDK bootstrapped; empty `dev` stack deploys
- [ ] Learning: TypeScript basics (types, interfaces, generics, async/await, modules)

**Done when:** a PR runs CI green and `main` deploys an empty `dev` stack.

## Phase 1 — Chat core (weeks 2–4)

- [ ] Cognito passwordless sign-in (email OTP)
- [ ] User profile (name, language, text size)
- [ ] Create family, invite code, join family
- [ ] Group conversations and 1:1 DMs
- [ ] Send / list messages (paged)
- [ ] Realtime delivery over WebSocket
- [ ] Push notifications for offline members
- [ ] Expo app: sign-in, family, chat list, chat screens (web + Android dev build)

**Done when:** two family members chat in realtime from web and phone.

## Phase 2 — AI (weeks 5–6)

- [ ] Bedrock adapter (ConverseStream), model IDs in config
- [ ] Bedrock Guardrail configured
- [ ] Private AI conversation
- [ ] `@ai` mentions in group chats
- [ ] Streaming replies to the app
- [ ] Per-user daily AI budget + usage tracking

**Done when:** a parent asks `@ai` a question in the family chat and sees the answer stream in.

## Phase 3 — Safety & media (week 7)

- [ ] Photo sharing (presigned upload, signed download)
- [ ] Scam checker: text input
- [ ] Scam checker: screenshot input
- [ ] Clear verdict card UI

**Done when:** a parent checks a real suspicious SMS and gets a clear verdict.

## Phase 4 — Parent polish & family beta (week 8)

- [ ] Large-text mode
- [ ] Voice input and read-aloud (on-device speech first)
- [ ] Onboarding designed for parents (≤ 3 steps)
- [ ] Prod deploy with all guardrails on
- [ ] Internal test build installed on parents' phones
- [ ] Collect feedback: what confused them, what they used most

**Done when:** your parents use it for a week without needing help.

## Later (not scheduled)

- Medication / bill reminders (EventBridge Scheduler)
- Shared family calendar and lists
- "Explain this letter / bill" from a photo
- Alert the family when a parent gets a likely scam
- More languages
- Public app store release
