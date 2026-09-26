# Kitchen Table: family games review and ideas

Date: 2026-09-26

Status: planning notes, not additional accepted architecture decisions

## Product direction

Kitchen Table is a private place for families and friend groups to play short games together. Chef is a game host. Table talk supports the games; it does not need to replace the family's existing chat app. The scam checker is out of scope entirely ([ADR-0012](../../docs/adr/0012-games-only-scope.md)).

The strongest product test is whether relatives come back to play with **each other**, not whether they use an AI feature. Design for a mix of ages, devices, and availability. The current 1–5 minute, asynchronous, cooperative principles in the [architecture](../../docs/ARCHITECTURE.md) are a useful default; verify them with actual players.

## Game ideas, in priority order

1. **Who Knows Us Best?** One person answers a light question such as “Which snack would I take on a road trip?” Others guess their answer. Reveal together, award points for matching, then let Chef add one short reaction. The family supplies the interesting content. Offer a curated question bank, skip, and “keep this answer private” controls. Avoid building a quiz from past personal answers without consent.
2. **Daily Question.** Everyone answers one prompt; answers appear after the player has answered or the day closes. Add a deadline so a missing player cannot hold the reveal forever. Chef can summarize only after the reveal and only from answers that participants agreed to share.
3. **Story Relay.** Chef opens a funny scenario. Each family member adds one sentence or chooses between two twists. The resulting story is a shareable family artifact. Keep turn and time limits so one inactive person does not block the group.
4. **Caption This.** A player contributes a photo or chooses a stock image; others submit captions and vote. Require the uploader's consent and a simple way to remove the image. This is a later game because private media handling adds work.
5. **Trivia Night.** Use a reviewed question bank for scored facts. Chef can host, explain, and add banter. Structured JSON validates an AI response's shape, but cannot establish that its stated answer is true. AI-generated scored questions need human review or reliable verification before release.
6. **Quick polls and “This or That.”** These are easy 30-second interactions between larger games. Prefer prompts about harmless preferences over sensitive family topics.
7. **Daily Word Puzzle and arcade.** The word puzzle is a good learning and engineering milestone, but it is largely solitary and may not validate the family-game promise. The arcade game adds a different play style, but should wait until the group loop proves fun.

For an initial family playtest, I would prototype **Who Knows Us Best?** or **Daily Question** with a room link, one prompt, answer/guess, reveal, and a replay button. Results can be shared in an existing chat app before building table talk, push, or a real-time service.

## Chef's role

- Generate optional variations of safe prompts, brief reactions, hints, and end-of-round recaps. Keep rules, score calculation, answer visibility, and deadlines in deterministic code.
- Make the game complete without a model response: use a curated prompt and a plain fallback message if AI is slow, unavailable, or over budget.
- Generate once per family/round and reuse the result. Avoid one paid model call per player action when one call serves the whole group.
- Do not reveal hidden answers, private submissions, or unshared family information through prompts or summaries. Make it clear when game content is sent to Chef.
- Use one consistent, warm tone. Teasing should never single out a child or turn a low score into embarrassment.

## Corrections and open decisions in the current documents

**Applied now**

- Removed scam-checker tasks and AI references from the current [architecture](../../docs/ARCHITECTURE.md) and [roadmap](../../docs/ROADMAP.md). Earlier ADRs remain as historical records; [ADR-0012](../../docs/adr/0012-games-only-scope.md) states the current scope.

**Resolve before implementing the relevant step**

1. **Group play mode.** The architecture excludes live real-time multiplayer, while the roadmap promises “Trivia Night” and streamed Chef messages. Specify whether v1 trivia is an asynchronous round with a closing time, or a scheduled live session. The latter changes presence, timing, reconnection, and test needs. WebSockets should be justified by a chosen experience rather than assumed for all games.
2. **Daily boundaries and reveals.** Define the family's timezone, the moment a daily prompt closes, what happens when someone skips or joins late, whether an answer can be edited, and whether unanswered members see others' responses at close. Use the same calendar-day rule for puzzles, streaks, and leaderboards. Step 1 needs an explicit date rule before family timezones exist.
3. **Cooperative scoring.** Define what a family streak requires and how absences or new members affect it. Avoid making one person responsible for breaking a streak. Consider “played together this week” as a gentler early milestone.
4. **Invitation and membership.** Decide whether a playtest can use a temporary room link before Step 2 sign-in. In production, redeem a single-use invite atomically and check family/game membership for every read and write, including WebSocket actions.
5. **AI job reliability.** SQS jobs can be retried or delivered more than once. Assign a stable game-event ID, make Chef's saved response and usage accounting idempotent, and define what clients display after a partial stream or reconnect. A saved final result must be retrievable without a live connection.
6. **Question accuracy.** Replace “AI-generated, validated trivia” with a precise quality rule. Schema validation checks format, not factual correctness. Scored trivia should use curated/reviewed answers until a verification process exists.
7. **Privacy and ages.** Confirm whether children will play. Set age-appropriate prompt and photo rules, explain who can see submissions, and give players a way to skip or delete personal content. The existing [no-E2E decision](../../docs/adr/0008-no-e2e-encryption-v1.md) should be translated into a plain-language notice before family beta.
8. **Roadmap proof point.** Step 1 currently proves a local word game. Add a small family-game playtest before investing in the full platform. Observe whether a group finishes a round, asks to replay, and understands reveal/scoring without help. Keep the word puzzle as a learning exercise if that goal remains important.

## Suggested next move

Choose the first **shared** game and write one page of rules: player count, invite flow, turn order, deadline, reveal, scoring, and what happens when someone leaves. Run a rough prototype with family. Let that result determine whether Step 2 should be accounts and leaderboards or a simpler shared room.
