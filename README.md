# Kitchen Table 🍽️

**Family game night, a few minutes a day.** A private place where a family plays small games
together — a daily word puzzle, a daily question, arcade mini-games, trivia — hosted by
**Chef**, an AI powered by Claude on Amazon Bedrock. Web, iOS and Android.

## Status

**Planning.** The architecture is agreed; building starts with Step 1, the Daily Word Puzzle.

## Docs

| Doc | What's in it |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | The system design: games, stack, data flow, data model, security |
| [Roadmap](docs/ROADMAP.md) | Build steps with checklists, from the first game to family beta |
| [Decision records](docs/adr/README.md) | Why each major choice was made |

## Stack at a glance

TypeScript everywhere · Expo (React Native) · Phaser · AWS Lambda + API Gateway · DynamoDB ·
Cognito (passwordless) · Amazon Bedrock (Claude) · EventBridge Scheduler · AWS CDK
