# Architecture Decision Records

Each file records one significant decision: the context, what we chose, and what it costs us.
To change a decision, add a new ADR that supersedes the old one — don't rewrite history.

| # | Decision | Status |
|---|---|---|
| [0001](0001-typescript-monorepo.md) | TypeScript monorepo (pnpm + Turborepo + CDK) | Accepted |
| [0002](0002-expo-universal-app.md) | Expo universal app; web + internal test builds first | Accepted |
| [0003](0003-serverless-backend.md) | Serverless backend: API Gateway + Lambda + Hono | Accepted |
| [0004](0004-dynamodb-single-table.md) | DynamoDB single-table design | Accepted |
| [0005](0005-cognito-passwordless.md) | Cognito passwordless auth | Accepted |
| [0006](0006-bedrock-ai-via-queue.md) | Bedrock AI calls through an SQS worker | Accepted |
| [0007](0007-single-aws-account.md) | Single AWS account with dev/prod stages | Accepted (revisit) |
| [0008](0008-no-e2e-encryption-v1.md) | No end-to-end encryption in v1 | Accepted |
| [0009](0009-english-first-i18n-ready.md) | English first, i18n-ready | Accepted |
| [0010](0010-games-first.md) | Games first: family game night, not another chat app | Accepted (scope clarified by 0012) |
| [0011](0011-game-logic-and-phaser.md) | Pure-TS game logic package + Phaser for arcade games | Accepted |
| [0012](0012-games-only-scope.md) | Family games only; remove the scam checker | Accepted |

Template: [`template.md`](template.md)
