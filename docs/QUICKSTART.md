# Quick setup overview

This public showcase contains no application source code. The complete source,
`.env.example`, and runnable setup are included with the Gumroad purchase.

## Prerequisites

- Node.js 20.9 or newer and npm.
- PostgreSQL 14+ or a compatible hosted PostgreSQL service.
- Docker Compose is optional for a local database.

## Setup flow

From the root of the purchased source project:

1. Install dependencies with `npm install`.
2. Copy `.env.example` to `.env`.
3. Set `DATABASE_URL`, `DIRECT_URL`, and a randomly generated `AUTH_SECRET`
   (at least 32 characters).
4. Generate Prisma Client with `npm run db:generate`.
5. Apply the development schema with `npm run db:push`, or apply committed
   migrations to a deployment database with `npm run db:deploy`.
6. Start the development server with `npm run dev`.

The app's landing pages and basic dashboard can run without third-party
credentials. Add provider credentials when enabling those integrations:

- Email sign-in: `RESEND_API_KEY`, `EMAIL_FROM`.
- Google OAuth: `AUTH_GOOGLE_ID`, `AUTH_GOOGLE_SECRET`.
- GitHub OAuth: `AUTH_GITHUB_ID`, `AUTH_GITHUB_SECRET`.
- Stripe billing: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`,
  `NEXT_PUBLIC_STRIPE_PRO_PRICE_ID`, and
  `NEXT_PUBLIC_STRIPE_LIFETIME_PRICE_ID`.

Never commit `.env` or expose server secrets in client-side variables. Refer to
the included English and Russian setup guides for the full instructions and
provider callback configuration.
