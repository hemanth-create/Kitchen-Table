# Kitchen Table 🍽️

**Short games that families and friends play together.** A private place where your group — a
*Table* — plays quick, turn-based games like **Who Knows Us Best?**, with an optional AI host,
**Chef** (Claude on Amazon Bedrock). Web first, then iOS and Android.

## Status

**Planning.** Next up: Step 0, a no-code WhatsApp playtest, then Step 1, the first game.

## Docs

| Doc | What's in it |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | The system design: games, stack, flows, data model, security |
| [Roadmap](docs/ROADMAP.md) | Build steps with checklists, from playtest to mobile apps |
| [Game rules](docs/games/who-knows-us-best.md) | Who Knows Us Best? — the first game |
| [Playtest kit](docs/games/whatsapp-playtest.md) | Run the game by hand in WhatsApp before building it |
| [Decision records](docs/adr/README.md) | Why each major choice was made |

## Stack at a glance

TypeScript everywhere · Expo (React Native) · AWS Lambda + API Gateway (Hono) · DynamoDB ·
Cognito (passwordless) · Amazon Bedrock (Claude) · EventBridge Scheduler · AWS CDK
