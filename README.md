# Launchpad — A Full-Scale SaaS Foundation

**Launch a SaaS product without assembling its core systems from scratch.**

Launchpad is a full-scale SaaS product foundation: one cohesive Next.js
codebase and shared PostgreSQL data model for payments, authentication,
analytics, administration, customer workspaces, content, and optional product
capabilities. It is built for teams that want control over their application
and infrastructure—not a hosted service or a collection of separately
deployed services.

> Launchpad includes production-oriented administration and security
> capabilities, but it does not make a product production-ready by itself.
> You remain responsible for provider configuration, deployment hardening,
> operational monitoring, and product-specific legal and security review.

[View Launchpad on Gumroad](https://shehroz2009.gumroad.com/l/launchpad-full-scale-saas-foundation)

## Product preview

![Launchpad public product experience](./screenshots/01-landing-dark.png)

| Customer workspace                                    | Pricing                                       |
| ----------------------------------------------------- | --------------------------------------------- |
| ![Customer dashboard](./screenshots/05-dashboard.png) | ![Pricing page](./screenshots/03-pricing.png) |

Browse the [English product screenshots](./screenshots/) or explore localized
landing and pricing pages for [all 18 locales](./docs/FEATURES.md#localization).

## What Launchpad includes

- **Customer accounts:** localized email sign-in, optional Google/GitHub OAuth,
  protected routes, TOTP multi-factor authentication, and recovery codes.
- **Billing:** Stripe subscriptions, one-time checkout, Customer Portal, signed
  webhooks, and optional Apple App Store / Google Play purchase verification.
- **Administration:** role-protected controls for users, security, analytics,
  integrations, content, support, and enabled modules.
- **Product workspace:** account settings, notifications, support, billing
  history, and optional organization and collaboration features.
- **Product capabilities:** optional modules for email campaigns, AI, publishing,
  marketing, storage, community, referrals, and more.
- **Operations:** guided first-run configuration, PostgreSQL and Prisma
  migrations, automated checks, and deployment documentation.

Provider-backed capabilities require your own provider accounts, credentials,
and configuration. The setup wizard can test selected credentials, but it does
not create accounts, complete OAuth consent, provision cloud infrastructure, or
verify every production permission.

See the [feature matrix](./docs/FEATURES.md) and the detailed
[implementation and operations guide](./docs/FEATURE-GUIDE.md) for current
scope, requirements, and limitations.

## How the application is structured

```mermaid
flowchart LR
  Visitor[Customer browser] --> App[Launchpad Next.js application]
  App --> Auth[Auth.js authentication]
  App --> DB[(Shared PostgreSQL via Prisma)]
  App --> Stripe[Stripe billing]
  Stripe --> Webhook[Verified provider webhooks]
  Webhook --> DB
  App --> Email[Resend, SendGrid, or SMTP]
  App --> Storage[Optional S3-compatible storage]
```

The application uses modular boundaries inside one deployable codebase.
Selected modules reuse the same application and database infrastructure;
module selection does not turn them into independent services or databases.

## Technology

| Area           | Technology                                                    |
| -------------- | ------------------------------------------------------------- |
| Application    | Next.js 16, React 19, TypeScript                              |
| Styling        | Tailwind CSS 4, reusable shadcn/ui-compatible components      |
| Authentication | Auth.js, email links, optional Google/GitHub OAuth            |
| Data           | PostgreSQL, Prisma                                            |
| Payments       | Stripe; optional Apple App Store and Google Play verification |
| Quality        | ESLint, Prettier, TypeScript checks, automated tests          |
| Deployment     | GitHub Actions checks; Vercel integration and host guides     |

## Start locally

Use Node.js 20.9 or newer (Node.js 22 LTS recommended) and PostgreSQL 14 or
newer. The included Compose file provides optional local PostgreSQL and
Mailpit services; it does not run the Launchpad application. Clone the
repository and run setup from the project directory:

```bash
git clone https://github.com/Shehroz2009/Launchpad.git
cd Launchpad
```

Then run the setup wizard:

```bash
npm run setup
```

The setup wizard configures the selected modules and deployment target, gathers
settings, and offers provider checks where supported. Read the
[quickstart guide](./docs/QUICKSTART.md) before configuring a real deployment.

## Documentation

- [Quickstart and configuration](./docs/QUICKSTART.md)
- [Feature matrix and current scope](./docs/FEATURES.md)
- [Implementation and operations guide](./docs/FEATURE-GUIDE.md)
- [Vercel deployment checklist](./docs/VERCEL.md)
- [Product screenshots](./screenshots/)

## License and purchase

Launchpad is licensed software for developing and operating one product or
service. The license does not permit redistributing Launchpad or its source.
Launchpad is actively maintained and receives ongoing product updates. Both
purchase plans include updates and direct support from the developer. Choose a
$129/month subscription, which includes access, updates, and support while
active, or a one-time $299 Lifetime purchase, which includes ongoing access,
updates, and support for the product's lifetime. Launchpad is application
software you run and operate, not a hosted SaaS service; hosting and third-party
provider fees are separate. Review the included [LICENSE](./LICENSE) for the
license terms.
