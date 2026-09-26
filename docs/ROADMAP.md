# Kitchen Table — Roadmap

Games first, learning one new thing per step. Each step ends with something your family can play.
See [ADR-0010](adr/0010-games-first.md) for why the order looks like this.

| Step | Build | You learn | AWS |
|---|---|---|---|
| 1 | Daily Word Puzzle | TypeScript, game state, input, testing | none |
| 2 | Families + daily leaderboard | Backend, database, auth, deploys | first contact |
| 3 | Daily Question + AI host "Chef" | Bedrock, schedules, realtime | yes |
| 4 | Arcade mini-game (Phaser) | Game loop, sprites, collisions | reuses leaderboard |
| 5 | Trivia, Who Knows Mom Best, table talk | Putting it all together | yes |

---

## Step 1 — Daily Word Puzzle (1–2 weeks, no AWS)

**Repo foundations**
- [ ] pnpm + Turborepo monorepo, strict shared `tsconfig`, ESLint, Prettier
- [ ] `packages/games` and `apps/mobile` (Expo, web target first)
- [ ] GitHub Actions: lint, typecheck, test on every push/PR

**Game rules (`packages/games`)**
- [ ] Puzzle number from date; word picked deterministically from a word list
- [ ] Guess scoring: correct / present / absent (handles repeated letters properly)
- [ ] Game state: guesses, win/lose, max 6 tries
- [ ] Share text: `Kitchen Table #N x/6` + emoji grid
- [ ] Unit tests for all of the above (Vitest)

**UI (`apps/mobile`, web)**
- [ ] Board with colored tiles, on-screen keyboard, physical keyboard support
- [ ] Save today's progress on the device
- [ ] "Copy result" button
- [ ] Large, high-contrast tiles (parent-friendly)

**Learning:** types, interfaces, union types, functions, modules, arrays/maps, testing.

**Done when:** your family plays the same word on the same day and shares results in WhatsApp.

## Step 2 — Families & leaderboard (2–3 weeks)

**AWS foundations**
- [ ] Root MFA on, IAM Identity Center user for daily work
- [ ] AWS Budgets alarm (e.g. $25/month) with email alert
- [ ] CDK bootstrapped; `dev` stage deploys from `main` via GitHub OIDC

**Features**
- [ ] Cognito passwordless sign-in (email OTP)
- [ ] User profile (name, language, text size)
- [ ] Create family, invite code, join family
- [ ] Word list moves server-side; guesses verified with `packages/games`
- [ ] Daily family leaderboard + family streak

**Done when:** everyone signs in and sees today's family leaderboard.

## Step 3 — Daily Question + Chef (2 weeks)

- [ ] Enable Bedrock model access for the chosen Claude models
- [ ] EventBridge Scheduler: daily drop per family (in family timezone)
- [ ] Question bank + AI-generated questions
- [ ] Answer-to-reveal rule enforced server-side
- [ ] Chef persona prompt, Bedrock Guardrail, per-family AI budget
- [ ] Chef's daily summary streamed over WebSocket
- [ ] Push notification: "Today's question is on the table 🍽️"

**Done when:** a parent answers the daily question and laughs at Chef's summary.

## Step 4 — Arcade mini-game (2–3 weeks)

- [ ] `apps/arcade` with Phaser 3 + TypeScript
- [ ] "Catch the Chapati": move, falling items, collisions, score, lives, difficulty ramp
- [ ] Embedded in the app (web directly, WebView on native)
- [ ] Family high-score board with server-side plausibility checks
- [ ] Android internal test build on parents' phones

**Learning:** game loop, delta time, sprites, input, collision, scenes, asset loading.

**Done when:** a family high-score rivalry starts.

## Step 5 — More games & table talk (3–4 weeks)

- [ ] Weekly Trivia Night (AI-generated, validated questions; weekly scoreboard)
- [ ] Who Knows Mom Best? (quiz built from past Daily Question answers)
- [ ] Table-talk threads per game/day, `@chef` mentions
- [ ] Weekly family recap from Chef
- [ ] Prod deploy with all guardrails on

**Done when:** your parents use it for a week without needing help.

## Later (not scheduled)

- Two Truths & a Lie, Caption This, collaborative AI-illustrated story
- AI cartoon versions of family photos
- Family Recipe & Story Book (voice → keepsake book)
- Reminders (medication, birthdays)
- More languages, public app store release
