# Kitchen Table — Roadmap

Prove the group loop first, then build the platform around it.
See [ADR-0013](adr/0013-shared-game-first.md) for why the order looks like this.

| Step | Build | Proves / teaches | AWS |
|---|---|---|---|
| 0 | WhatsApp playtest of Who Knows Us Best? | Do people want to play? | none |
| 1 | Who Knows Us Best? — rules + local web prototype | TypeScript, game state, API, testing | none |
| 2 | Deploy the prototype with room links → real playtest | Serverless, DynamoDB, CDK, CI/CD | first contact |
| 3 | Accounts, multiple Tables, invites | Auth, permissions | yes |
| 4 | Chef + Daily Question + push | Bedrock, schedules, notifications | yes |
| 5 | Story Relay, quick polls, mobile builds | More games; native apps | yes |
| — | Optional learning track: Word Puzzle, Phaser arcade | Game loop, sprites, collisions | reuses leaderboard |

---

## Step 0 — WhatsApp playtest (5–7 days, no code)

- [ ] Run [the playtest kit](games/whatsapp-playtest.md) in one family or friend group
- [ ] Record participation, replay requests, confusion and ideas each day
- [ ] Decide: build Step 1, adjust and re-test, or try Daily Question instead

**Done when:** you have a written go / adjust / stop decision.

## Step 1 — Who Knows Us Best? prototype (2–3 weeks, no AWS)

**Repo foundations**
- [ ] pnpm + Turborepo monorepo, strict shared `tsconfig`, ESLint, Prettier
- [ ] `packages/games`, `packages/shared`, `packages/core`, `packages/adapters`, `apps/api`, `apps/mobile`
- [ ] GitHub Actions: lint, typecheck, test on every push/PR

**Game rules (`packages/games`)** — per [the rules page](games/who-knows-us-best.md)
- [ ] Rotation (join order, sit out, pass), question selection without repeats
- [ ] Round state: open → picks → revealed (lazy: all submitted or deadline passed)
- [ ] Scoring, known-by score, weekly board, "played together" milestone
- [ ] Table-timezone day/week boundaries with an injected clock
- [ ] Curated question bank (JSON, 50+ questions, human-reviewed)
- [ ] Unit tests for every rule above

**API + app (local)**
- [ ] Hono API on Node with in-memory adapters; room link + device token
- [ ] Server never returns picks before the reveal (tested)
- [ ] Web screens: join via link, round, pick, reveal, next round, weekly board
- [ ] Large, clear UI; works on a phone browser

**Learning:** types, unions, interfaces, generics, modules, async/await, testing, HTTP APIs.

**Done when:** 3+ players (separate browser profiles or phones on your Wi-Fi) play full rounds end to end.

## Step 2 — Deploy and playtest (1–2 weeks)

**AWS foundations**
- [ ] Root MFA on, IAM Identity Center user for daily work
- [ ] AWS Budgets alarm (e.g. $25/month) with email alert
- [ ] CDK bootstrapped; `dev` stage deploys from `main` via GitHub OIDC

**Deploy**
- [ ] DynamoDB adapters (conditional writes for open-round lock and picks)
- [ ] API on Lambda + API Gateway (throttling on); web app on S3 + CloudFront
- [ ] Plain-language privacy note in the app
- [ ] Playtest with 1–2 real groups for a week; track the product metrics

**Done when:** a real group finishes rounds without help and asks to replay.

## Step 3 — Accounts & Tables (2 weeks)

- [ ] Cognito passwordless sign-in (email OTP; passkeys later)
- [ ] User can sit at multiple Tables (family, friends…)
- [ ] Single-use invite codes, redeemed atomically
- [ ] Playtest players claim their existing seats
- [ ] Membership check on every read/write (tested)

**Done when:** you sit at a family Table and a friends Table from one account.

## Step 4 — Chef, Daily Question, push (2–3 weeks)

- [ ] Write the Daily Question rules page first
- [ ] Enable Bedrock model access; Chef persona prompt + Guardrail
- [ ] `ai-worker`: idempotent event IDs, timeout, fallback text, per-Table budget
- [ ] Chef reaction after each reveal (optional, never blocking)
- [ ] Daily Question with EventBridge Scheduler (Table timezone), lazy reveal at close
- [ ] Push notifications: "new round", "results are in" (Expo Push)

**Done when:** Chef's reactions make people laugh, and the game still works with Chef switched off.

## Step 5 — More games & mobile (3–4 weeks)

- [ ] Rules pages first: Story Relay, quick polls
- [ ] Story Relay (turn + time limits so one person can't block the group)
- [ ] Quick polls / This or That
- [ ] Android internal test build (and TestFlight if needed)
- [ ] Prod deploy with all guardrails on

**Done when:** groups play at least twice a week for a month.

## Optional learning track (any time after Step 1)

- [ ] Daily Word Puzzle in `packages/games` (solo, shareable result)
- [ ] Phaser arcade mini-game with a Table high-score board

## Later (not scheduled)

- Caption This (private media, uploader consent, easy removal)
- Trivia Night with a human-reviewed question bank
- Family Recipe & Story Book
- More languages, public app store release
