# Quickstart

Launchpad is an application you run and operate, not a hosted service. You
provide its runtime, database, provider accounts, and production credentials.

## Requirements

- Node.js 20.9 or newer; Node.js 22 LTS is recommended.
- npm.
- PostgreSQL 14+; Docker Desktop or another Docker Compose-compatible runtime
  is optional if you want to use the included local PostgreSQL and Mailpit.

## First run

Clone the repository before setup. The included Compose configuration provides
optional local PostgreSQL and Mailpit services; it does not run the Launchpad
application. After setup, the wizard tells you whether a local service is
needed and which Compose command to use.

```bash
git clone https://github.com/Shehroz2009/Launchpad.git
cd Launchpad
```

Run the setup wizard:

```bash
npm run setup
```

The guided wizard lets you choose a deployment target and optional modules,
collects settings, and offers provider checks for supported credentials. Keep
the terminal open and follow the local URL printed by the wizard.

Setup is a one-time operation. Applying module selection removes unselected
module source from the working tree through Git sparse-checkout; the source
remains in the repository and can be restored later. Setup requires a clean
Git working tree and does not support downloaded ZIP archives or copied source
folders.

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
