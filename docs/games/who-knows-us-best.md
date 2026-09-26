# Who Knows Us Best? — Game rules (v1)

> The first shared game. Rules here are implemented in `packages/games` as pure, deterministic
> code. Chef (AI) never decides anything in this document — it only reacts.

## In one line
One player secretly picks their answer to a light question; everyone else guesses it; all
answers are revealed together.

## Players
- A **Table** is a private group — a family or a friend group. Players are adults.
- A round needs **at least 3 active players** (1 subject + 2 guessers). Max 12 per Table in v1.

## A round
1. **Open.** Any player taps *New round*. The game picks:
   - the **subject**: next player in the rotation (see below), and
   - a **question** from the curated bank, not used at this Table before
     (repeats only after the bank is exhausted).
2. **Submit (all at once, all hidden).**
   - The subject picks **their true answer** from 4 options.
   - Every other player picks **what they think the subject chose**.
   - Anyone can change their pick until the reveal.
3. **Reveal** happens at whichever comes first:
   - every active player has submitted, or
   - the **deadline**: 24 hours after the round opened.
4. **Results.** Everyone sees the subject's answer, who guessed what, and the
   "known-by" score (e.g. "3 of 4 know Dad's road-trip snack"). Players who didn't submit
   still see the results.
5. **Next round.** Any player can start the next round right away (max 10 rounds per Table per day).
   Only one round is open per Table at a time.

## Question format
- Light, harmless preferences and habits — e.g. *"Which snack would I take on a road trip?"*
  with 4 options.
- 4 fixed options (no free text in v1: no fuzzy matching, no AI judging).
- Questions come from the **curated bank** only. Categories: food, travel, habits,
  would-you-rather, nostalgia. No questions about health, money, relationships, politics or religion.

## Subject rotation
- Round-robin in join order.
- A player can **sit out** (skipped in the rotation and not counted as active) and come back anytime.
- The subject can **swap the question once** per round, or **pass** (the next player becomes subject).

## Scoring
- Each correct guess: **+1 point** for the guesser.
- The subject scores nothing — being well-known is shown as the *known-by* score, not points,
  so nobody is rewarded for being predictable or punished for being surprising.
- **Weekly board:** "Best guesser this week" (Monday–Sunday in the Table's timezone).
- **Table milestone (cooperative):** "Played together this week" — at least 3 rounds in the week
  where 50%+ of active players took part. Never names who missed.

## Time rules
- Every Table has a **timezone**, set by its creator (defaults to their device's timezone).
- "Day" and "week" always use the Table's timezone.
- Deadlines are stored as absolute UTC timestamps.
- A round is **revealed lazily**: whenever it is read, if everyone has submitted or the deadline has
  passed, it is treated as revealed. No background job is needed for the reveal itself.

## Joining and leaving
- A new player joins from the **next** round and is added to the end of the rotation.
- If a **guesser** leaves mid-round, their pick is removed.
- If the **subject** leaves mid-round, the round is cancelled with no points.
- Removing a player removes their picks from open rounds; revealed history keeps an anonymous
  "former player" label.

## Privacy
- A player's pick is visible only after the reveal, and only to that Table.
- Answers are used **only in that round**. They are never reused in other games, quizzes or AI
  prompts unless the player opts in (not in v1).
- Any player can delete their own picks from history.

## Chef (optional)
- After the reveal, Chef may post **one** short reaction for the round. One AI call per round,
  never per player.
- Chef only sees revealed data: the question, the subject's answer, the known-by score, first names.
- If the AI is slow, unavailable or over budget, a plain message is shown instead
  ("3 of 4 of you know Dad well! 🍽️"). The game never waits on Chef.
- Warm, playful tone; teasing is about the answer, never about a person's score.

## Example
> **Question for Mom:** Which snack would I take on a road trip?
> A) Chips B) Fruit C) Murukku D) Chocolate
>
> Mom picks **C**. Dad guesses C ✅, Priya guesses D ❌, Arjun guesses C ✅.
> **Reveal:** "2 of 3 know Mom's road-trip snack." Dad +1, Arjun +1.
> **Chef:** "Murukku on the highway — a classic. Priya, chocolate was a bold guess. 🍫"
