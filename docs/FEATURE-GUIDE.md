# Launchpad feature overview

This guide is a concise overview of the Launchpad starter kit for prospective
buyers. It summarizes what is included, what requires external services or
configuration, and what the kit does not provide automatically. The purchased
project's full feature guide and source code remain the authoritative
implementation references.

## What Launchpad includes

Launchpad is a modular SaaS application built on one shared codebase. Its
optional modules cover billing, AI chat, organizations, publishing, support,
community, notifications, email marketing, analytics, referrals, advertising,
SEO, integrations, and private file storage. The setup wizard lets you choose
which optional modules to retain and a deployment target. Core account,
security, settings, and shared database components remain in place.

In a Git checkout, setup uses sparse-checkout so excluded tracked files remain
in Git history. In a purchased ZIP, excluded module files are removed from the
extracted project; they cannot be restored from Git history because the ZIP
does not contain repository metadata. This is a one-time setup, not an
in-place module-switching tool. Back up the extracted ZIP before configuring
it if you may want to recover excluded source later.

The wizard writes selected modules and deployment preferences to
`launchpad.config.json`, and provider secrets to the Git-ignored `.env` file.
It can check selected provider credentials when you explicitly choose **Test
connection**. It does not create provider accounts, deploy infrastructure,
or configure external dashboards. You still need to complete and maintain
those services yourself.

The plugin area exposes an allow-listed set of bundled components; it is not
an arbitrary package marketplace and does not download or execute uploaded
plugin code. The optional event-relay service can be deployed separately for
the currently supported account-profile update event. It is not a general
service registry, message bus, or per-consumer acknowledgement system.

## Accounts and security

Authentication supports email sign-in and optional Google and GitHub sign-in.
In development, Mailpit can receive sign-in mail when no email provider is
configured. Deployed environments need an email provider for email sign-in
and transactional mail. Admin pages and APIs enforce roles on the server.

Optional TOTP multi-factor authentication works with compatible
authenticator apps and includes one-time recovery codes. Security settings
include session management and audit activity. The kit does not include
password-based sign-in or a password-recovery flow.

The account-onboarding checklist covers profile settings, account security,
and the dashboard; it is optional and can be revisited.

## Billing and paid access

The billing module supports Stripe Checkout for configured subscriptions and
one-time Lifetime purchases, plus the Stripe Customer Portal. Optional Apple
App Store and Google Play purchase verification can provide another billing
source when the required products, credentials, store accounts, and native
client integration are configured. Store routes are unavailable until their
settings are provided; Stripe can continue to operate independently.

The kit processes verified payment and subscription events, records payment
history, and handles refunds and disputes. It does **not** automatically
protect every feature in your product just because billing is enabled.
`paymentBlocked` is a billing signal, not a universal paywall. Each paid
feature must enforce access on the server; hiding a button in the browser is
not sufficient. Decide which features are paid and what restricted customers
can still do, then implement and test those rules. The kit also does not
include a ready-made administrator workflow for verifying payments made
outside Stripe.

Billing email uses the configured email provider, but billing does not
require email to function. Email delivery can be delayed, fail, or be
duplicated after an uncertain network response; it is not an exactly-once
delivery system.

## Email, waitlist, and marketing

The shared email transport supports Resend, SendGrid, and SMTP. It is used
for authentication, billing notices, notifications, newsletters, and
marketing messages. Sender identities and provider credentials must be
configured. The setup wizard's SMTP check authenticates without sending a
message.

The localized early-access waitlist requires explicit consent. Its statuses
are managed manually: inviting or marking someone as joined does not create
an account, send an invitation, or grant access. Waitlist consent is separate
from newsletter subscription.

The newsletter supports consent-based signups, unsubscribe links, and
administrator-created plain-text campaigns. Marketing automation supports
short email sequences for new newsletter subscribers. These tools are not a
visual email editor, audience segmentation system, or compliance/legal
service. They do not automatically enroll waitlist entries or send messages
through push notifications and webhooks. Scheduled campaigns and automation
require an external scheduler as well as a configured email provider.

## AI assistant and organizations

The optional dashboard assistant connects to a configured OpenAI-compatible
text API. It stores conversations in the application database, enforces
account request limits, and supports personal and organization conversations.
It does not include a model, execute tools, or take autonomous actions.
Prompts and responses are sent to the configured provider and stored by the
application; configure the provider's privacy and retention settings and
disclose this processing to users.

Organizations provide shared workspaces, member roles, and invitations.
Authorization is checked against organization membership, but this is
shared-database tenancy, not a separate database or schema for each customer.
Billing, account settings, support, notifications, analytics, newsletters,
and global publishing content are not automatically organization-scoped.

## Publishing, support, and community

Separate modules provide a localized Markdown blog, release-update feed, and
CMS pages. Administrators manage drafts and published content. Public pages
include localized metadata and sitemap support. The publishing tools do not
include scheduled publishing, automatic translation, image uploads, or a
comment-moderation system.

The support module provides customer tickets and administrator replies. The
community module provides a discussion forum. The curated directory lets
administrators manage public entries. These are starter features rather than
integrations with a hosted help desk or a full social network: attachments,
assignment/SLA automation, forum reactions, accepted answers, and public user
profiles are not included.

## Notifications and customer data

The dashboard has a persistent notification inbox and optional email, Web
Push, and signed-webhook delivery channels. Push notifications require VAPID
keys and browser subscription; webhook consumers must verify signatures and
deduplicate events. Automatic retries for notifications, marketing, and
storage cleanup need externally scheduled jobs.

Account settings provide a JSON export of supported account data and a
separate link to billing management. The export excludes credentials,
sessions, signing secrets, and other sensitive tokens. It is a streamed
download, not a full database backup or a background archive service.

## Analytics, localization, and customization

The dashboard includes account and billing views. Optional product analytics
collect consent-gated events and provide administrative reports; they are
not a replacement for a third-party analytics platform. Attribution,
marketing tracking, and referral cookies may have consent obligations in
your deployment. Launchpad does not include a consent-management platform.

The interface supports 18 locales: English, Arabic, Bengali, German,
Spanish, French, Hindi, Indonesian, Italian, Japanese, Korean, Dutch, Polish,
Portuguese, Russian, Turkish, Vietnamese, and Chinese. English is the
fallback catalog and the setup wizard's fallback language. Review translations
and legal text for your audience before launch.

The product builder edits an allow-listed set of application sections,
navigation, localized copy, and brand settings. It does not generate
arbitrary code, create application routes, or change business logic. Theme,
motion, and other personal interface preferences are stored in the browser,
not synchronized across devices.

## Storage, referrals, advertising, and SEO

Optional private file storage uses S3-compatible object storage. The
application validates file types and sizes, uses short-lived upload/download
URLs, and checks file ownership. Configure a private bucket and the required
credentials. The module does not scan files for malware, and account deletion
does not automatically remove remote objects.

The referral module tracks partner links and Stripe-derived commissions.
Payouts are performed outside Launchpad. It does not calculate commissions
for App Store or Google Play purchases. Referral attribution uses a
30-day cookie; Launchpad has no built-in consent manager, so assess and
implement applicable consent requirements.

Advertising tools organize campaign and account information and import
reports from CSV. They do not connect to ad networks to create or manage live
campaigns. SEO features provide localized metadata, sitemaps, robots rules,
structured data, and an application-database audit; they are not a web
crawler, ranking tracker, keyword research tool, or redirect manager.

The integration catalog reports whether expected configuration is present.
It does not prove that credentials work, create provider accounts, or
guarantee delivery.

## Deployment and operating responsibilities

Deployments require PostgreSQL and appropriate environment variables,
including database connections and `AUTH_SECRET`. Configure email, Stripe,
store billing, object storage, AI, push, and scheduled jobs only for the
features you enable. Production migrations must be applied using the
documented deployment process; keep preview environments on an isolated
database.

The application includes server-side validation, role checks, security
controls, and audit logging, but it is not a complete security operations
program. It does not provide a WAF, malware scanning, external security
monitoring, or a strict Content Security Policy. Configure a trusted ingress
and hosting protections, review the threat model, and test the deployed
application before launch.

For detailed configuration, data ownership, limits, migrations, and
operational procedures, see the full feature guide shipped with the project.
