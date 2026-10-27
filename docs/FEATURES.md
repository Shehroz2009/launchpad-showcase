# Feature matrix

Launchpad is a full-scale SaaS product foundation with modular capabilities
that run in one application and share its PostgreSQL data model. This matrix
describes the current implementation, not externally provisioned provider
accounts, production infrastructure, or credentials. Integrations require
setup in the provider and in your deployment environment.

## Product capabilities

| Area                      | Current implementation and boundaries                                                                                                                                                                                                                                             |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AI                        | Optional OpenAI-compatible assistant, per-account limits, private conversations, export/deletion, and organization-shared conversations; requires a provider key.                                                                                                                 |
| Admin panel               | Separate admin sections for users, analytics, content, integrations, security, support, and other enabled modules; pages and APIs enforce server-side admin authorization.                                                                                                        |
| Analytics                 | Consent-aware product events, account identification after login, account-aware funnels and anonymous sessions, weekly cohorts, and configurable retention. Cohorts are browser-session based; no third-party analytics service is bundled.                                       |
| Authentication            | Auth.js magic-link sign-in through Resend, SendGrid, or SMTP; optional Google/GitHub OAuth; JWT sessions, localized route protection, TOTP MFA, and recovery codes.                                                                                                               |
| Billing                   | Stripe subscriptions and lifetime entitlements, Customer Portal, signed webhooks, and idempotent subscription/payment processing; requires Stripe setup.                                                                                                                          |
| Blog                      | Localized admin publishing, drafts, safe Markdown/GFM, language-specific slugs, and sitemap integration.                                                                                                                                                                          |
| CMS                       | Separate localized public-page editor with drafts, summaries, SEO/indexing controls, and sitemap integration. It does not edit arbitrary application code or routes.                                                                                                              |
| Checkout                  | Server-created Stripe Checkout sessions for subscription and lifetime purchases; a browser redirect alone never grants an entitlement.                                                                                                                                            |
| UI components             | Reusable Button, Card, Input, and Badge foundation with Tailwind and shadcn/ui-compatible styling; not a large component marketplace.                                                                                                                                             |
| Customization             | Landing-page sections and copy, CTA destinations, navigation and dashboard widgets, and shared brand color/corner settings. Does not edit arbitrary screens, components, routes, or business logic.                                                                               |
| Themes                    | Light, dark, and system modes; browser-local accent presets/custom color, contrast checks, corner radius; admins can set shared brand defaults.                                                                                                                                   |
| Customer dashboard        | Responsive workspace with account/plan information, billing history, settings, notifications, support, and enabled feature areas.                                                                                                                                                 |
| Database                  | PostgreSQL, Prisma schema/client, migrations, Docker Compose for local development, and setup/seed commands. Production database configuration is required.                                                                                                                       |
| Email                     | Shared adapter for Resend, SendGrid, or custom SMTP; transactional sign-in/billing mail, consent-based newsletter, resumable broadcasts, and scheduled signup sequences. Provider setup is required. SMTP checks do not send mail and cannot guarantee provider-side idempotency. |
| Integrations              | Admin catalog reports configured/missing settings without exposing secrets; setup wizard tests selected supported credentials. It does not create accounts or automatically authorize every provider.                                                                             |
| Landing and pricing pages | Localized public pages; the product builder can edit supported sections, copy, visibility, order, and safe CTA destinations.                                                                                                                                                      |
| Payments                  | Stripe billing plus App Store/Google Play purchase verification; requires corresponding provider credentials and configuration.                                                                                                                                                   |
| SEO                       | Localized metadata editor, audits of stored public content, canonical/hreflang, Open Graph/Twitter, JSON-LD, robots.txt, and sitemap. No ranking/impression analytics, live-site crawler, keyword research, or search-engine automation.                                          |
| Security                  | TOTP MFA, recovery codes, rate limits, session revocation, audit/security monitor, input validation, signed webhooks, and baseline security headers. No bundled WAF, automated vulnerability monitoring, or strict nonce-based CSP.                                               |
| Settings                  | Account profile, language/time zone, security, data export, notifications, theme, UI preferences, and animation preferences. Some appearance preferences are browser-local rather than synced across devices.                                                                     |
| Storage                   | Optional private S3-compatible object storage with short-lived URLs, ownership checks, and file limits; requires storage setup.                                                                                                                                                   |
| User management           | Admin user search, role/status filters, pagination and counts; role changes and suspend/reactivate actions include safeguards for the last admin and self-demotion.                                                                                                               |
| Multi-tenancy             | Organizations, memberships, invitations, roles, and organization-scoped AI/files. Tenants share application infrastructure and database; separate databases/cloud accounts are not provisioned.                                                                                   |
| Responsive design         | Responsive customer-facing and admin layouts with mobile navigation.                                                                                                                                                                                                              |
| Localization              | 18 routed locales with locale-aware navigation, metadata, fallback, and RTL support. Translation quality and completeness can vary; product-builder copy may fall back to English where localized copy is unavailable.                                                            |
| Notifications             | Persistent inbox plus opt-in email, signed user webhook, and browser push; channel retries are independent. Built-in event generation covers billing events, not arbitrary app-wide routing rules.                                                                                |
| Deployment                | Setup guidance for Docker/VM, AWS ECS, Kubernetes, Cloud Run, and Fly.io; Vercel production builds apply migrations. Does not provision cloud resources or generate automated deploy workflows for every host.                                                                    |
| Animations                | Browser-local controls for supported entrance style, timing/easing, distance, stagger, hover, and reduced motion. Applies to wired components/shared tokens; no per-element timeline editor.                                                                                      |
| Referral tracking         | Referral links, first-touch attribution, partner applications, Stripe-based commission ledger, and admin approval/payment tracking. Payouts occur outside the app; refunds/store commissions are not automatically reconciled.                                                    |
| Support                   | Private account-owned tickets, replies, categories, status/priority, admin responses, and private staff notes; no external support vendor required.                                                                                                                               |
| Directory                 | Localized, searchable public listings with categories/tags, featured entries, and admin publication/indexing controls.                                                                                                                                                            |
| UI preferences            | Browser-local density, content width, sticky navigation, transparency, and focus-ring controls; no arbitrary screen/layout editor.                                                                                                                                                |
| Navigation                | Localized responsive navigation with protected areas; the product builder can reorder/hide allowlisted entries but does not create arbitrary routes.                                                                                                                              |
| Onboarding                | Account-owned resumable checklist for profile, security, and dashboard orientation.                                                                                                                                                                                               |
| Plugins                   | Admin marketplace installs/toggles reviewed source-bundled plugins with signed installation state. Arbitrary uploads, marketplace downloads, and execution of buyer/database-provided code are not supported.                                                                     |
| CI/CD                     | GitHub Actions validates pull requests and `main`; Vercel Git integration can deploy previews/production, and production builds run migrations. No universal release orchestrator or workflow pack for every host.                                                                |
| Microservices             | Transactional event outbox, cursor-based read-only API, and separately deployable relay; currently emits the account-profile event. The application is one deployable system, not a set of independently owned domain services.                                                 |
| Marketing                 | UTM links, attributed public-page/CTA/newsletter/checkout reporting, and scheduled email sequences for new newsletter signups. No arbitrary analytics-event triggers/actions or cross-channel behavioral automation.                                                              |
| Community                 | Localized forum with topics/replies, abuse reports, admin moderation, pinning, and locked threads.                                                                                                                                                                                |
| Product updates           | Localized versioned changelog with drafts, publishing, Markdown, and sitemap integration; separate from blog/CMS.                                                                                                                                                                 |
| Ads                       | Local ad-account/campaign records, UTM links, CSV metric imports, and comparison with attributed events. No network OAuth, live synchronization, or control of external ads.                                                                                                      |
| In-app purchases          | Server verification and account binding for Apple App Store and Google Play, lifecycle notifications, and native client examples; requires store credentials.                                                                                                                     |
| Waitlist                  | Localized opt-in signup with separate consent and an admin queue for search/status/deletion; independent of newsletter delivery.                                                                                                                                                  |

## Localization

The application routes these 18 locales: English, Arabic, Bengali, German,
Spanish, French, Hindi, Indonesian, Italian, Japanese, Korean, Dutch, Polish,
Portuguese, Russian, Turkish, Vietnamese, and Chinese. Catalog key coverage is
tested, but that does not guarantee every custom content field or product
builder string is translated. English fallback can be visible when localized
content is not configured.

The showcase includes English product screens, Russian and Arabic dashboard
examples, and landing/pricing screenshots for each non-English locale:

| Locale            | Landing                                     | Pricing                                        |
| ----------------- | ------------------------------------------- | ---------------------------------------------- |
| Arabic (`ar`)     | [Screenshot](../screenshots/30-home-ar.png) | [Screenshot](../screenshots/31-pricing-ar.png) |
| Bengali (`bn`)    | [Screenshot](../screenshots/32-home-bn.png) | [Screenshot](../screenshots/33-pricing-bn.png) |
| German (`de`)     | [Screenshot](../screenshots/34-home-de.png) | [Screenshot](../screenshots/35-pricing-de.png) |
| Spanish (`es`)    | [Screenshot](../screenshots/36-home-es.png) | [Screenshot](../screenshots/37-pricing-es.png) |
| French (`fr`)     | [Screenshot](../screenshots/38-home-fr.png) | [Screenshot](../screenshots/39-pricing-fr.png) |
| Hindi (`hi`)      | [Screenshot](../screenshots/40-home-hi.png) | [Screenshot](../screenshots/41-pricing-hi.png) |
| Indonesian (`id`) | [Screenshot](../screenshots/42-home-id.png) | [Screenshot](../screenshots/43-pricing-id.png) |
| Italian (`it`)    | [Screenshot](../screenshots/44-home-it.png) | [Screenshot](../screenshots/45-pricing-it.png) |
| Japanese (`ja`)   | [Screenshot](../screenshots/46-home-ja.png) | [Screenshot](../screenshots/47-pricing-ja.png) |
| Korean (`ko`)   | [Screenshot](../screenshots/48-home-ko.png) | [Screenshot](../screenshots/49-pricing-ko.png) |
| Dutch (`nl`)      | [Screenshot](../screenshots/50-home-nl.png) | [Screenshot](../screenshots/51-pricing-nl.png) |
| Polish (`pl`)     | [Screenshot](../screenshots/52-home-pl.png) | [Screenshot](../screenshots/53-pricing-pl.png) |
| Portuguese (`pt`) | [Screenshot](../screenshots/54-home-pt.png) | [Screenshot](../screenshots/55-pricing-pt.png) |
| Russian (`ru`)    | [Screenshot](../screenshots/56-home-ru.png) | [Screenshot](../screenshots/57-pricing-ru.png) |
| Turkish (`tr`)    | [Screenshot](../screenshots/58-home-tr.png) | [Screenshot](../screenshots/59-pricing-tr.png) |
| Vietnamese (`vi`) | [Screenshot](../screenshots/60-home-vi.png) | [Screenshot](../screenshots/61-pricing-vi.png) |
| Chinese (`zh`)    | [Screenshot](../screenshots/62-home-zh.png) | [Screenshot](../screenshots/63-pricing-zh.png) |

## Architecture and quality

Launchpad uses Next.js 16, React 19, strict TypeScript, Tailwind CSS 4,
PostgreSQL/Prisma, and Auth.js v5. Selected modules share the same application
and database infrastructure. The repo also configures ESLint, Prettier, Husky,
lint-staged, automated tests, and GitHub Actions.

See [QUICKSTART.md](./QUICKSTART.md) for setup and
[FEATURE-GUIDE.md](./FEATURE-GUIDE.md) for detailed behavior, data flows,
provider requirements, and operational boundaries.
