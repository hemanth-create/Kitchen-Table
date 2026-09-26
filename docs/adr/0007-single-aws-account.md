# ADR-0007: Single AWS account with dev/prod stages

- **Status:** Accepted (revisit before public launch)
- **Date:** 2026-09-26

## Context
There is one existing AWS account. Separate accounts (via AWS Organizations) give the
strongest isolation, but add setup. At family scale, simplicity wins for now.

## Decision
Use **one account** with two CDK stages, `dev` and `prod`, and these guardrails:
- Every resource name prefixed with its stage.
- Prod DynamoDB: deletion protection + point-in-time recovery.
- Prod S3: `RETAIN` removal policy + versioning.
- Prod CloudFormation stacks: termination protection.
- Separate IAM deploy roles per stage; AWS Budgets alarm on the account.
- Root user MFA, no daily root use; IAM Identity Center for logins.

## Consequences
- A mistake in `dev` shares quotas (e.g. Bedrock throughput) and the bill with `prod`.
- **Revisit:** move to AWS Organizations with separate `dev`/`prod` accounts before any
  public launch or when others join development. Data at family scale is small enough to migrate.
