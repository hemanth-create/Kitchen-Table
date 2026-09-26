# Kitchen Table 🍽️

A private family chat app — web, iOS and Android — with an AI assistant at the table.

Built for families, designed for parents: group chat, an `@ai` helper powered by Claude on
Amazon Bedrock, and a one-tap **"Is this a scam?"** checker.

## Status

**Planning.** No application code yet — the architecture is agreed first, then built phase by phase.

## Docs

| Doc | What's in it |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | The system design: stack, data flow, data model, security |
| [Roadmap](docs/ROADMAP.md) | Phases and checklists, from foundations to family beta |
| [Decision records](docs/adr/README.md) | Why each major choice was made |

## Stack at a glance

TypeScript everywhere · Expo (React Native) · AWS Lambda + API Gateway · DynamoDB ·
Cognito (passwordless) · Amazon Bedrock (Claude) · AWS CDK
