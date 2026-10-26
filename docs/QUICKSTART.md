# Quickstart

Launchpad is an application you run and operate, not a hosted service. You
provide its runtime, database, provider accounts, and production credentials.

## Requirements

- Node.js 20.9 or newer; Node.js 22 LTS is recommended.
- npm.
- Docker Desktop or another Docker Compose-compatible runtime for the included
  local PostgreSQL service, or a PostgreSQL 14+ database.

## First run

Start the local database before setup if you plan to use Docker Compose. From a
clean project archive or checkout, run:

```bash
docker compose up -d
npm run setup
```

The guided wizard lets you choose a deployment target and optional modules,
collects settings, and offers provider checks for supported credentials. Keep
the terminal open and follow the local URL printed by the wizard.

Setup is a one-time operation. Applying module selection removes unselected
module source from the extracted project. In particular, an archive has no Git
history from which to recover removed files; keep an untouched copy if you may
need other modules later. In a clean Git checkout, setup requires a clean
working tree and applies sparse-checkout selection.

The wizard does not create provider accounts, complete interactive OAuth
authorization, or provision cloud resources. Provider checks validate a
specific operation only, not every permission or production setting.

## Run the application

After the wizard completes, follow its configuration output, then run:

```bash
npm run dev
```

The application uses the configured PostgreSQL database and applies the
project's normal Prisma workflow. For production, use the deployment platform's
secret manager for credentials and apply committed database migrations before
serving traffic.

## Integrations

The exact required values depend on selected modules. Start with the generated
`.env` and `.env.example`; never commit credentials or expose server secrets in
`NEXT_PUBLIC_*` variables.

| Capability       | Typical configuration                                             |
| ---------------- | ----------------------------------------------------------------- |
| Database         | PostgreSQL `DATABASE_URL` and `DIRECT_URL`                        |
| Authentication   | A strong `AUTH_SECRET` and public application URL                 |
| Email            | Resend, SendGrid, or SMTP credentials and sender identity         |
| OAuth            | Credentials and registered callback URLs for Google and/or GitHub |
| Stripe           | API key, webhook signing secret, and configured price IDs         |
| AI               | An OpenAI-compatible provider endpoint and API key                |
| Object storage   | S3-compatible endpoint, bucket, and least-privilege credentials   |
| Mobile purchases | App Store / Google Play application and verification credentials  |

Provider credentials are optional until their features are enabled. Email
sign-in, payments, OAuth, AI, storage, and mobile purchase verification need
provider-specific setup to work in a real environment.

## Before production

Configure production secrets and database access; verify OAuth callbacks,
email deliverability, payment price IDs and webhooks, storage permissions,
backups, privacy/consent settings, and the operational limits of each provider.
See the [full feature guide](./FEATURE-GUIDE.md) and
[Vercel checklist](./VERCEL.md).
