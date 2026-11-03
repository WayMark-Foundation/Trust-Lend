# TrustLend

> Consent-based onboarding, screening, scoring and lending for people with no credit history.

**Status:** early development. This is a final-year project and "TrustLend" is a working name.
It uses synthetic data only, is not a licensed lender, and moves no real money.

## What it does

TrustLend helps a "thin-file" borrower, someone with steady income but no credit score, get a fair loan decision.

- Verifies identity from an ID document and screens the name against public watchlists.
- Reads bank statements only with the applicant's consent, through a simulated Account Aggregator.
- Produces an explainable score and plain-language reasons.
- Runs the loan through offer, acceptance, repayment and reminders.
- Sends unclear cases to a human officer and records every step in a tamper-evident audit log.

## Architecture

TrustLend is a distributed system. Each service runs as its own container, owns its own data, and talks to the others only through REST APIs and events.

| Service | Language and framework | Role |
|---|---|---|
| `identity` | Java, Spring Boot | Login, roles, tokens |
| `kyc` | TypeScript, Express (plus a Python OCR worker) | ID intake, OCR, watchlist screening |
| `consent` | TypeScript, Express | Consent records, statement fetching |
| `scoring` | Python, FastAPI | Features, score, reasons |
| `lending` | TypeScript, Express | Offers, ledger, repayments, notifications, AI assistant |
| `review-audit` | Java, Spring Boot | Officer review queue, hash-chained audit log |
| `sim` | TypeScript, Express | Mock bank and mock Account Aggregator (test props) |
| `web` | TypeScript, Next.js | Applicant, officer and admin interfaces |

Shared infrastructure: PostgreSQL, Redis, RabbitMQ, MinIO.

## Repository layout

```
services/   one folder per service, each with its own build, tests and Dockerfile
sim/        simulated bank and Account Aggregator
web/        Next.js front end
packages/   small shared TypeScript libraries (auth, http, events, config, logging)
contracts/  OpenAPI files and event schemas, the agreed boundaries between services
ml/         notebooks, training and evaluation (datasets are not committed)
deploy/     local docker-compose and Helm skeleton
docs/       architecture notes, decision records (ADRs), threat model, runbooks
tools/      scripts and synthetic data generators
```

## Why a monorepo, for now

Where the code lives is a separate decision from how the system runs. The services are independent at runtime, and we keep the code in one repository because, for a four-person team, it is the simplest way to keep them consistent:

- A change to an API or event often touches several services, and one pull request keeps them in step.
- One CI setup, one issue board and one set of review rules.
- Contracts and shared tooling live in one place, and reviewers can see the whole system.

Independence is enforced, not assumed:

- Each service has its own folder, dependencies, tests, Dockerfile and Helm release.
- Each service has its own database schema and database user. No service reads another's tables.
- Services never import each other's code. They may use only `packages/`, and CI fails if the rule is broken.
- Services communicate only through the contracts in `contracts/`.
- CI runs only for the folders that changed, so a change to one service builds and deploys only that service.

We will split a service into its own repository if it needs a separate team, a separate release cycle or separate access control. Because the boundaries are already enforced, that move is a cheap folder extraction.

## Getting started

Local setup instructions will be added with the first walking skeleton.
