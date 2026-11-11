# Quickstart

Launchpad is an application you run and operate, not a hosted service. You
provide its runtime, database, provider accounts, and production credentials.

## Requirements

- Node.js 20.9 or newer; Node.js 22 LTS is recommended.
- npm.
- PostgreSQL 14+; Docker Desktop or another Docker Compose-compatible runtime
  is optional if you want to use the included local PostgreSQL and Mailpit.

## First run

Clone the repository before setup.

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

## Local development

After the wizard completes, follow its configuration output, then run:

```bash
npm run dev
```

The application uses the configured PostgreSQL database and applies the
project's normal Prisma workflow. This starts a local development server; it does not publish the application.

## Production container deployment

The repository root contains a multi-stage `Dockerfile`. Build the application
image with the public URL that will be used in production, and build the
separate migration image:

```bash
docker build --build-arg NEXT_PUBLIC_APP_URL=https://app.example.com -t launchpad:app .
docker build --target migration -t launchpad:migrations .
```

Push both images to your provider's registry. Configure production values in
the provider's secret manager, including `DATABASE_URL`, `DIRECT_URL`,
`AUTH_SECRET`, `AUTH_URL`, and `NEXT_PUBLIC_APP_URL`; run the migration image
once before routing traffic to the application image on port `3000`. Do not
pass server secrets to `docker build` or bake them into the image.
`NEXT_PUBLIC_*` values are fixed at build time, so provide any required public
values before building. After migrations are applied, `GET /api/health` returns
HTTP 200 when PostgreSQL is reachable and HTTP 503 when it is not.

This is a manual container workflow, not an automated provisioning wizard.
You still create the provider resources, configure the domain/TLS and any
scheduled jobs, and arrange image builds, registry pushes, and releases.
Vercel deployments use the separate [Vercel checklist](./VERCEL.md).

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
