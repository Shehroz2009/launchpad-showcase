# Feature guide

This guide describes the behavior of the modules currently included in
Launchpad, where their data lives, and which configuration is needed. It
documents the starter implementation; it is not a claim that provider
accounts, production secrets, or third-party dashboards are preconfigured.

## First-run project setup

Extract the purchased ZIP archive, open a terminal in the extracted project
directory, and run `npm run setup` before installing dependencies or
customizing the application. A Git repository is not required. The wizard also
supports a clean Git checkout, provided it is not already using sparse-checkout.
In either case, the directory must not contain `launchpad.config.json`. The
command starts a temporary HTTP server bound only to `127.0.0.1` on a random
port and prints a tokenized URL in the terminal. Open that URL on the same
computer, keep the terminal open, and do not share the URL. The wizard supports
English and Russian.

Choose one deployment target, optional modules, and any provider values you
already have. The selected target and module IDs are written to
`launchpad.config.json`; secret values stay in the local, Git-ignored `.env`.
Core email-provider values remain available because authentication and
transactional mail use them even when email campaigns are not selected.
Setup also removes npm commands whose source entry points were excluded.
`npm test` discovers only tests physically present in the configured checkout;
CI uses the same command rather than invoking removed module scripts. Support
admin routes and ticket components are owned by the support module, and
dashboard shortcuts disappear when their target module is disabled. CI builds
isolated core-only and selected-module configurations with freshly installed,
pruned dependencies. Run `node scripts/verify-module-build.mjs core-only` or
`node scripts/verify-module-build.mjs selected` to repeat those checks locally.
For an unconfigured checkout, `selected` exercises billing, support, and
plugins; after setup it uses the saved module selection.
For an unconfigured distribution, CI also runs
`node scripts/verify-module-build.mjs archive-core`: a fresh directory without
Git or dependencies starts the real bootstrap/wizard, rejects unauthorized
configuration access, applies a core-only selection over HTTP, verifies persisted
pruning, then installs, tests and builds the result. The archive wizard scenario
is intentionally omitted for already-configured checkouts.
The main CI job migrates disposable PostgreSQL and supplies
`BILLING_TEST_DATABASE_URL`, `NOTIFICATION_TEST_DATABASE_URL` and
`SECURITY_TEST_DATABASE_URL` to `npm test`. Database scenarios exercise first-time
Auth.js adapter registration, event commit/cursor ordering, checkout retries,
refunds/clawbacks, notification leases, analytics ownership, storage races,
account exports, campaign audience snapshots and concurrent quotas. This verifies
local business invariants, not live OAuth/store/payment-provider configuration.
The wizard creates a random `AUTH_SECRET` if the existing value is missing,
placeholder text, or shorter than 32 characters. Provider APIs are not called
automatically. An explicit **Test connection** action sends only the entered
key and required settings to the selected provider: Stripe uses the read-only
account endpoint, OpenAI-compatible AI requests the configured `/models`
endpoint, Resend lists API keys, and SendGrid uses Mail Send sandbox mode (no
email is delivered). A successful Resend key-list check proves API-key access,
not email-sending permission; a sending-only Resend key may receive a warning
because it cannot list keys. Stripe's account read checks authentication but
does not confirm test/live mode or every permission required by the app.
Applying setup requires an in-memory HMAC proof for the exact submitted values;
editing a key, sender, or endpoint invalidates the corresponding client check,
and the server independently enforces the proof before writing `.env`. The
proof is neither written to disk nor logged. OAuth credentials cannot be
validated without completing their interactive authorization flow; other
provider settings still need provider-specific setup. The wizard does not
provision infrastructure or generate platform-specific deployment workflows.
The platform choice is a saved preference with a checklist, not an automated
deployment.

The wizard preserves literal dollar signs, backslashes, quotes, and multiline
credentials when writing `.env`. Values containing carriage returns or all three
quote delimiters (`"`, `'`, and backtick) are rejected explicitly instead of being
silently changed; provide these unusual values through the deployment environment.

After checking local imports, the wizard prunes unused runtime dependencies,
updates `package-lock.json`, and keeps only the selected module source. In a
Git checkout it enables sparse-checkout, so excluded tracked files remain in
Git history. In the purchased ZIP, excluded module files are removed from the
extracted project; they are not recoverable from Git history because the
archive has no repository metadata. Shared code used by core flows, all
Prisma schema and migration history, documentation, and development/build
dependencies are retained. Therefore `node_modules` may remain substantial:
Next.js, Prisma, TypeScript, and shared runtime dependencies are not optional.
The Vercel configuration retains only cron schedules whose modules are
selected.

This is a one-time bootstrap: the wizard refuses to rerun after a successful
setup. For Git checkouts it also refuses to run with a dirty working tree.
In a ZIP distribution, back up the extracted folder before setup if retaining
unselected module source is important. Sparse-checkout patterns cover the files
present at setup time; if a later upstream update adds a file that should be
included, add that path with `git sparse-checkout add <path>`. Changing module
selections after customization is not an in-place wizard operation; preserve
your changes and start again from an untouched purchased archive or clean
checkout.

## Authentication and account access

Auth.js v5 is configured in `src/auth.ts` with a Prisma adapter and JWT
sessions. Email sign-in is enabled when the selected email provider key is set. Its
single-use verification link expires after 15 minutes; the email is localized
using the locale of the sign-in page. Google and GitHub appear only when both
credentials for that provider are configured.

The middleware redirects unauthenticated requests for localized dashboard and
admin pages to the matching login page. Admin pages and admin APIs also check
the user's role on the server; hiding a link in the UI is not treated as
authorization. A suspended account is rejected by the Auth.js sign-in callback.
Profile updates are validated on the server and only modify the signed-in
user's record.

Account security settings support optional TOTP multi-factor authentication
(MFA) with any compatible authenticator app. Setup is confirmed with a live
six-digit code before MFA is enabled. The TOTP secret is encrypted at rest
using `AUTH_SECRET`; ten one-time recovery codes are returned once and stored
only as keyed hashes. A recovery code is consumed atomically when used.

After the first sign-in factor, each JWT session has its own database-backed
MFA verification state. Until that session completes MFA, dashboard pages and
account/admin APIs protected by `getCurrentUser()` do not authorize it; the
localized `/[locale]/mfa` challenge accepts either a TOTP or recovery code.
The MFA endpoints are rate-limited per account, and enable/disable/verification
actions are recorded in the security audit log. Disabling MFA requires a valid
unused TOTP code and invalidates recovery codes and all existing JWT sessions,
requiring a fresh sign-in. Each accepted TOTP time step is consumed atomically
per account to prevent reuse across sessions; re-enabling requires verification
again on other sessions. This uses the existing Auth.js providers and does not
require an external MFA service or additional environment secret.

The account-security API is split into `POST /api/account/security/mfa/setup`
(generate and encrypt a pending secret), `POST
/api/account/security/mfa/enable` (confirm TOTP and issue recovery codes),
`POST /api/account/security/mfa/verify` (complete a pending login session), and
`POST /api/account/security/mfa/disable` (require TOTP before removal). Session
records older than 90 days are pruned when a new MFA session is created; an
older still-active login must verify again if it is used afterward.

Apply the `20261024140000_account_mfa` migration before deploying. Recovery
codes are shown only after enabling and cannot be retrieved later; users should
save them securely. If an account loses both its authenticator and every
recovery code, an administrator must use the application's account recovery
process rather than reading or recovering stored MFA secrets.

The admin Security center at `/{locale}/admin/security` summarizes security
events and rate-limit buckets from the last 24 hours, active-account MFA
coverage, and the most recent 100 audit events. Failed TOTP and recovery-code
checks are recorded without storing submitted codes. Rate-limit rows contain a
control scope and threshold alongside an HMAC-derived key; the dashboard groups
exceeded buckets without exposing the underlying account identifier or IP
address. Apply `20261024200000_security_monitoring` to enable this monitoring.
These are application-level signals, not infrastructure telemetry, a WAF,
malware scanner, or an external alerting/SIEM integration.

## Account onboarding

The localized checklist at `/[locale]/dashboard/onboarding` guides signed-in
users through three optional first steps: review profile settings, review
account security, and explore the dashboard. The overview shows the checklist
until it is completed or dismissed; the dashboard sidebar keeps a direct link
so dismissed or completed progress can be revisited. No marketing, billing,
notification, support, or external provider integration is required.

Apply the `20261023150000_onboarding_progress` migration before deployment.
`OnboardingProgress` stores a unique row per user with completed step IDs,
completion and dismissal timestamps. `PATCH /api/account/onboarding` accepts
only the fixed step allowlist, rejects duplicates and unknown values, and
scopes changes to the authenticated account. Marking a step complete clears
dismissal; completing all steps records completion, while choosing to skip
hides the overview prompt without deleting progress. The account JSON export
includes onboarding progress, and deleting a user cascades the row.

## Plugins

The admin-only marketplace at `/[locale]/admin/plugins` manages a curated
catalog of first-party dashboard extensions. It currently includes account
preferences (enabled by default), notifications, and support shortcuts. Each
entry has a validated ID/version/slot manifest and a component explicitly
imported in `src/plugins/registry.ts`; installation, enablement, update
acknowledgement, and removal are controlled from the interface.

Marketplace changes persist only an allowlisted plugin ID, version, install
state, and enable state. An HMAC signature derived from `AUTH_SECRET` covers
that state; dashboard loading verifies it with a constant-time comparison and
checks the exact ID and version against the source-controlled registry.
Unexpected, altered, or unavailable entries are quarantined and never loaded.
The only executable code is the server-rendered component already bundled in
the application. Nothing is downloaded from a marketplace, evaluated from a
database, uploaded by users, or installed as a runtime dependency. New plugin
code and versions require a reviewed application deployment; the UI cannot
publish new code or bypass that release boundary.

Apply the `20261024150000_plugin_marketplace` migration. Marketplace
installations are global to the application, not per-user. The existing account
preferences widget remains installed by default; additional plugins can be
installed, enabled, disabled, or removed independently. Plugin registry
validation rejects invalid manifests, duplicate IDs, and missing slot
implementations. Installation changes are recorded in the security audit log.
`GET /api/admin/plugins` returns the verified catalog; its POST endpoint accepts
only the actions `install`, `enable`, `disable`, `update`, and `uninstall`.
“Update” acknowledges a version already included in the deployed application;
it does not download a package. Keep `AUTH_SECRET` stable or valid install
records will be quarantined and need to be reinstalled.

## Independently deployable services

The app remains a modular monolith by default. Its optional PostgreSQL
transactional outbox lets separately deployed consumers process account
events without making profile updates depend on those consumers. Apply the
`20261023160000_service_events` migration and set a unique server-only
`SERVICE_API_KEY` of at least 32 characters to enable the feed. If the key is
unset, event writes and the feed are disabled while account flows continue to
work.

`GET /api/internal/events?afterId=0&limit=100` accepts a `Bearer` key and
returns up to 100 events in ascending numeric-ID order. The initial
`account.profile.updated` event is committed in the same transaction as the
profile change. Its payload contains the event type and changed field names,
not profile values. All publishers must use `recordServiceEvent` inside their
transaction: it holds a shared publication advisory lock until commit before
allocating an ID, preventing a cursor from skipping a lower ID committed later.
Consumers should save their string cursor after handling
an event and deduplicate by event ID; delivery can repeat after a crash.
Deleting an account nulls the event's user reference, and the account JSON
export includes events still associated with that account. The endpoint is
read-only and uses `Cache-Control: no-store`. The key grants access to the
whole feed, so keep it server-side, distribute it only to trusted consumers,
rotate it if exposed, and use TLS.

`services/event-relay` is a dependency-free Node.js worker that can be deployed
separately from the web application. It polls the feed, forwards each event to
one configured HTTP service, signs the raw JSON body with HMAC-SHA256, and
persists its cursor only after the receiver returns a success status. Failed
deliveries retry without advancing the cursor. This is at-least-once delivery:
receivers must deduplicate using `X-Launchpad-Event-Id` and should verify
`X-Launchpad-Signature` over `<timestamp>.<raw request body>` with
`SERVICE_EVENT_WEBHOOK_SECRET`; reject stale timestamps. Keep the cursor volume
persistent across restarts. A duplicate may still occur if delivery succeeds
but cursor persistence fails.

Build the standalone image from the repository root:

```sh
docker build -f services/event-relay/Dockerfile -t launchpad-event-relay services/event-relay
docker run --rm \
  -e SERVICE_EVENT_SOURCE_URL \
  -e SERVICE_API_KEY \
  -e SERVICE_EVENT_TARGET_URL \
  -e SERVICE_EVENT_WEBHOOK_SECRET \
  -v launchpad-event-relay-state:/var/lib/launchpad \
  launchpad-event-relay
```

Set those variables in the deployment secret store or shell environment; do
not bake credentials into the image. Both keys must be at least 32 characters.
HTTPS is required except for
loopback development URLs. `SERVICE_POLL_INTERVAL_MS` defaults to 1000 and
accepts values from 250 to 60000 milliseconds. Each relay instance has one
cursor and one target; do not share its cursor volume with another instance.
Independent consumers can poll the source API directly and maintain their own
cursors. There is no runtime service registry, producer push delivery,
per-consumer acknowledgement in the source feed, or automatic event retention.
Only the profile event is currently emitted; add versioned event contracts and
tests when introducing additional domain events.

The admin user tools support case-insensitive name/email search, role and
status filters, 20-user pages, and workspace-wide total/active/suspended/admin
counts. Admins can change another user's role or suspend/activate the account.
The last administrator cannot be demoted, and administrators cannot revoke
their own access.

## Billing and transactional email

The pricing page uses Stripe Checkout for the configured monthly Pro price and
one-time Lifetime price. The server creates the Checkout session using the
authenticated account; a browser redirect never grants access. Stripe
Customer Portal handles subscription self-service.

Apply migration `20261022133000_checkout_reservations` before deploying Checkout.
One durable `CheckoutAttempt` per account freezes the price, plan, original
locale, redirect base URL and customer details. Both the attempt and customer
link commit before a payable session is created; a lost response retries the
same attempt key rather than creating another purchase. PostgreSQL locks
serialize requests across application instances. Requests for the same plan
reuse the open URL, including requests in another locale (the original locale
remains in redirects/receipts). A plan/price change first expires the old
session and confirms provider expiration before committing a new attempt.
Sessions expire after one hour. Near-expired attempts with an unknown session
are recovered from Stripe before replacement; time alone never releases an
ambiguous purchase. Completed/pending payments block another purchase until
the signed webhook commits fulfillment. Failed/expired purchases can be retried,
and account/store/subscription eligibility is rechecked on the server.
For upgrades from an installation predating reservations, disable Checkout,
expire legacy open sessions in Stripe, and reconcile any completed asynchronous
payments before enabling the new endpoint; legacy sessions have no attempt ID.

`/api/stripe/webhook` validates Stripe signatures and reads at most 1 MiB of
payload before verification. Subscription, invoice,
payment, and processed-event writes are made transactionally and webhook event
IDs make repeated deliveries safe. The email module adds a transactional
outbox: successful one-time/recurring payments queue receipts, failed invoices
queue payment-failure notices, and delivery is retried when Stripe redelivers
the event. Resend idempotency keys prevent repeated provider sends. Checkout
locale is copied into Stripe metadata so billing emails can be sent in English
or Russian. Email delivery uses the selected provider; billing itself does not
require an email provider.

`sendTransactionalEmail` in `src/lib/email.ts` sends through the shared
`EmailAdapter` interface in `src/lib/email-provider.ts`. Set
`EMAIL_PROVIDER=resend` (the backward-compatible default) with
`RESEND_API_KEY`, `EMAIL_PROVIDER=sendgrid` with `SENDGRID_API_KEY`, or
`EMAIL_PROVIDER=smtp` with `EMAIL_SMTP_HOST` and the SMTP settings below.
All providers use the same adapter for authentication, billing, notifications,
newsletters, and marketing.

Outbound Resend, SendGrid and SMTP delivery has an absolute eight-second
deadline. Resend/SendGrid receive an abort signal; SMTP destroys its actual
socket at the deadline, in addition to its inactivity timeout. SendGrid error
bodies are limited to 1,000 bytes. Signed notification webhooks and Web Push
also have an eight-second deadline and close their request/socket resources.
Campaign and automation batches contain at most ten recipients with five
concurrent sends: their worst-case network budget is 16 seconds, leaving
headroom for database work within the 60-second HTTP job. Timeout errors remain
failures, not successful deliveries; provider acceptance before a lost response
is still ambiguous and SMTP/SendGrid remain at-least-once.

For SMTP, `EMAIL_SMTP_PORT` defaults to 587, `EMAIL_SMTP_SECURE=false` uses
STARTTLS, and `EMAIL_SMTP_REQUIRE_TLS=true` requires STARTTLS. Set
`EMAIL_SMTP_SECURE=true` for implicit TLS (commonly port 465); in that mode
`EMAIL_SMTP_REQUIRE_TLS` is ignored. `EMAIL_SMTP_USER` and
`EMAIL_SMTP_PASSWORD` are optional but must be configured together. For a
trusted internal relay that intentionally does not support TLS, set
`EMAIL_SMTP_REQUIRE_TLS=false`; credentials should not be sent over plaintext.
The local setup wizard's SMTP check connects, negotiates TLS as configured, and
authenticates without sending a message. SMTP cannot guarantee provider-side
idempotency: delivery retries after an uncertain network failure may duplicate
a message. SendGrid `custom_args` are tracing metadata, not deduplication.
Billing outbox messages use an atomic five-minute delivery lease to prevent
concurrent sends; only the owning worker can mark the message sent. Expired
leases can be retried. A crash after provider acceptance but before saving `SENT`
can still duplicate mail with SMTP or SendGrid: this is at-least-once, not
exactly-once delivery. The integration catalog reports configuration presence, not live
deliverability.

Marketing automation claims also carry a unique ownership token: a recovered
worker cannot have its progress overwritten by a delayed former worker, and
unsubscribe invalidates an in-flight claim. Apply migration
`20261024230000_marketing_delivery_fencing` before deploying the worker update.
An email already accepted by the provider cannot be recalled by unsubscribe;
SMTP/SendGrid retries still have the at-least-once limitation described above.
Newsletter campaign recipients use the same ownership fencing after migration
`20261024231000_campaign_delivery_fencing`: a stale failure cannot reset a
recipient already marked sent by a replacement worker.
Stop or drain the old campaign/automation workers before deploying these
updates; do not run old unfenced workers alongside the new version.

## Early-access waitlist

The localized public form is available at `/[locale]/waitlist`. It requires
explicit consent specific to early-access updates; that consent does not
subscribe the address to the newsletter. The API validates the email and
locale, normalizes email casing, records the consent timestamp, and returns a
generic success for duplicate signups without changing their existing status.
A hidden honeypot helps discard basic automated submissions.

Administrators manage the latest 500 entries at
`/[locale]/admin/waitlist`: search by email, mark an entry `WAITING`, `INVITED`,
or `JOINED`, and permanently delete it. Older entries remain in storage and
can still be managed through database operations. Status changes are manual;
they do not create accounts, send invitations, or grant access. The module
does not depend on the newsletter, email provider, user registration, or
Marketing analytics. Newsletter and waitlist requests have separate application-wide
budgets of 120 attempts/minute and 3 attempts/email/hour; signup bodies are limited
to 2 KiB. Marketing and product analytics each allow 600 attempts/minute across the
application, plus 60 events/authenticated account/minute. These atomic database
budgets include invalid requests and cannot be bypassed by rotating client session
IDs or forwarding headers. Exhaustion returns HTTP 429, so a noisy client can
temporarily consume a shared budget. Add per-client ingress/WAF limits and request
timeouts before exposing these endpoints publicly; only trust network identifiers
overwritten by your ingress, and prevent direct access to the origin.
Each account may own at most 10 organizations and register at most 10 Web Push
endpoints. Transactional locks enforce both quotas under concurrent requests;
refreshing an existing push endpoint remains allowed at capacity. Organization
creation allows 10 attempts/hour/account (2 KiB body), and push registration allows
60 attempts/hour/account (4 KiB body). Quota exhaustion returns HTTP 409; request
rate exhaustion returns HTTP 429.

Apply migration `20261024100000_waitlist_entries` before deployment. The
`WaitlistEntry` table stores a unique normalized email, selected language,
consent timestamp, status, and record timestamps; it does not store IP
addresses or user-agent strings.

## App Store and Google Play purchases

In-app purchases are an optional billing source alongside Stripe. The
`20261018040000_in_app_purchases` migration adds account-link identifiers,
verified purchase records, and an idempotency ledger for store notifications.
Run `npm run db:deploy` before enabling these API routes.

Configure store product IDs with the comma-separated
`APPLE_PRO_PRODUCT_IDS`, `APPLE_LIFETIME_PRODUCT_IDS`,
`GOOGLE_PLAY_PRO_PRODUCT_IDS`, and `GOOGLE_PLAY_LIFETIME_PRODUCT_IDS`
allowlists. Apple verification also needs `APPLE_BUNDLE_ID`,
`APPLE_ROOT_CERTIFICATES_BASE64` (comma-separated base64 DER Apple root
certificates), and `APPLE_ENVIRONMENT`; production additionally requires the
numeric `APPLE_APP_ID`. Google verification needs
`GOOGLE_PLAY_PACKAGE_NAME` and `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`, a service
account with Android Publisher API access. These provider settings are optional;
an unconfigured store returns a clear 503 while Stripe and other modules remain
available.

An authenticated native client calls `GET /api/iap/account` to obtain the
user's stable Apple `appAccountToken` and pseudonymous Google Play account ID.
StoreKit should attach the UUID when purchasing; Play Billing should set the
returned ID as its obfuscated account ID. The client sends a StoreKit signed
transaction JWS, or the Play product ID/type and purchase token, to
`POST /api/iap/verify`. The server verifies Apple's signature, bundle, product
allowlist, and account token, or fetches the purchase directly from the Google
Android Publisher API and verifies its product and obfuscated account ID.
Google tokens are encrypted with AES-256-GCM at rest; a SHA-256 hash supports
lookup without storing the raw token as an index. The server acknowledges
unacknowledged Play purchases after verifying account ownership.

App Store Server Notifications v2 are signature-verified at
`/api/iap/apple/notifications` with a 128,000-byte request body limit. Google Real-time Developer Notifications are
accepted at `/api/iap/google-play/notifications` as Pub/Sub push messages with
Google-signed OIDC tokens. Configure `GOOGLE_PLAY_PUBSUB_AUDIENCE` and
`GOOGLE_PLAY_PUBSUB_SERVICE_ACCOUNT_EMAIL` to match the authenticated push
subscription. Signature, expiration, audience and verified email are checked; the server
re-queries Google for the current status rather than trusting notification
state. Both webhook paths record event IDs transactionally, reject account
reassignment, and make duplicate deliveries safe. Apple signed event times
prevent older notifications from overwriting newer subscription state.
Apply migration `20261022151000_google_purchase_state` before deployment.
Authenticated Google voided notifications persist a token-hash tombstone without
calling the purchase API, including when notification delivery precedes client
registration. Later verification cannot revive a revoked or superseded token.
Subscription replacements use Google's `linkedPurchaseToken` to locate the
previous registered account, then independently verify the new product and
obfuscated account ID. Saving a replacement retires the old token atomically;
an old token first registered later is also denied access. Unknown non-voided
tokens with no registered predecessor return a retryable error, not `200 ignored`:
configure Pub/Sub retries/dead-letter monitoring and register purchases promptly.
Google notification package names must match the configured app. Android
Publisher reads/acknowledgements have eight-second timeouts with no implicit retries.

Verified store entitlements appear on dashboard/billing and block duplicate
Stripe checkout while active. Stripe records and Customer Portal behavior are
otherwise unchanged; store subscriptions are managed by their respective
stores. `examples/in-app-purchases/ios/StoreKitPurchaseClient.swift` and
`examples/in-app-purchases/android/PlayPurchaseClient.kt` demonstrate the
native request flow; each app must add its own authenticated HTTP client and
store product setup.

## AI assistant

The assistant is an optional dashboard feature backed by an OpenAI-compatible
Chat Completions endpoint. Set the server-only `AI_API_KEY` to enable it.
`AI_BASE_URL` defaults to `https://api.openai.com/v1`, `AI_MODEL` defaults to
`gpt-4o-mini`, and `AI_SYSTEM_PROMPT` optionally supplies an application-level
system message. The configured endpoint must use HTTPS except for loopback
development URLs. No AI provider SDK is required; the adapter uses the standard
HTTP API shape, so compatible providers can be configured without tying the
application to a single vendor.

Apply the `20261022000000_ai_assistant` migration before deployment. User
conversations and messages are stored in `AiConversation` and `AiMessage`.
Personal conversations belong to the authenticated account; organization
conversations are visible only to members of the selected organization.
`GET /api/ai/chat` returns only the caller's personal or authorized
organization conversation list and one conversation's messages.
`POST /api/ai/chat` validates a prompt, verifies personal ownership or
organization membership, sends the recent context to the provider, and stores
the completed user/assistant turn transactionally.
`DELETE /api/ai/chat?conversationId=...` deletes only the caller's personal
conversation or an organization conversation the caller created (unless the
caller is an organization administrator). The account JSON export includes
personal conversation content only.

Requests are limited to 20 per authenticated account per fixed 24-hour
window starting with its first request, using the AI module's atomic database
counter. This is not a rolling window: requests can burst around a reset boundary.
A prompt is capped
at 8,000 characters, context at the 12 latest messages, provider output at
1,024 tokens, and each provider request at 30 seconds. Provider errors are
returned as a generic 502 and logged without response bodies, prompts, or
credentials. Missing configuration returns 503 for chat requests and hides the
navigation link, while billing, notifications, and the rest of the dashboard
continue to work.

Prompts and completions are sent to the configured provider and stored in the
application database. The operator is responsible for consent, privacy
disclosures, provider retention settings, and any applicable regulatory
requirements. This module is a text assistant only: it does not execute tools,
perform autonomous actions, or supply a model.

## Multi-tenancy

Apply the `20261024170000_multi_tenancy` migration before deployment.
Signed-in users create organizations at `/[locale]/dashboard/organizations`.
The creator becomes the owner; owners can invite administrators or members,
administrators can invite members, and members can leave. Invitation tokens
are random 256-bit values stored only as SHA-256 hashes, expire after seven
days, and are accepted only by a signed-in account whose email matches the
invitation. Hexadecimal tokens are case-insensitive and normalized before hashing.
Invitations are delivered through the selected email provider;
if email delivery is not configured, an authorized organization manager receives the
one-time link to share through a trusted channel.
Owners cannot leave or be removed; ownership transfer and organization
deletion are intentionally not exposed.

The dashboard workspace selector switches between personal data and one
organization. The selection is a client preference, not an authorization
credential: each API request checks the authenticated user against
`OrganizationMembership` before reading or writing organization data. Roles
are `OWNER`, `ADMIN`, and `MEMBER`; member management, organization renaming,
invitation creation, and revocation are authorized server-side.
These management writes lock the organization and recheck current membership
inside the same transaction. A manager removed by a concurrent request cannot
continue a waiting management operation with stale permissions. Repeated member
removal returns not-found rather than an unhandled database exception.
Invitation acceptance uses the same lock order as management operations and
rechecks revocation and expiration after waiting for an in-flight change.
AI exchanges and new file upload records recheck organization membership in the
write transaction under a shared organization lock. A member removed while a
request is in flight, including during AI generation, cannot write afterward;
a conversation deleted during generation returns 404 instead of a server error.

AI conversations and stored files have an optional `organizationId`. Personal
records remain owned by one account; organization records are readable by
members, and only their creator or an organization administrator can delete
them. S3-compatible object keys use an organization-specific prefix, and
personal and organization storage have separate 100-file quotas. AI request
limits remain per account. Account exports include only that account's
personal conversations and files, not shared organization records.

This is shared-database tenancy using explicit foreign keys and membership
checks, not separate database/schema provisioning. Billing, subscriptions,
account security/preferences, notifications, support tickets, analytics,
newsletter records, and global blog/CMS content retain their existing
account-wide or application-wide ownership. The module adds no requirement
for AI or object storage: organizations and memberships work without either;
the existing features continue to work in personal mode without an
organization.

## Blog publishing

The blog is a publishing module, separate from the general-purpose CMS. Each
localized post is an independent `BlogPost` row with its locale, unique
locale-scoped slug, excerpt, Markdown body, tags, draft/published status, and
publication timestamps. This lets the blog be deployed and used without
enabling the CMS; it does not introduce CMS page types or a shared
content-storage dependency.

Apply the `20261023000000_blog` migration before deployment. Administrators
manage the latest 100 posts under `/en/admin/blog` or `/ru/admin/blog`.
Admin-only `GET` and `POST /api/admin/blog`, `PATCH
/api/admin/blog/[postId]`, and `DELETE /api/admin/blog/[postId]` validate
inputs on the server. Slugs use lowercase ASCII letters, digits, and hyphens;
the same slug can be used once in each locale. Publishing sets
`publishedAt`; returning a post to draft clears it. Duplicate locale/slug
pairs return HTTP 409.

Public `/[locale]/blog` indexes and `/[locale]/blog/[slug]` articles show only
published posts whose publication date has arrived. The sitemap includes
those posts and their update times. Page metadata includes canonical URLs,
descriptions, tags, Open Graph article fields, and BlogPosting structured data.
Posts render CommonMark/GFM through `react-markdown` and `remark-gfm`; raw HTML
is not enabled, so stored post bodies are not injected as HTML. Markdown image
URLs render as links rather than causing automatic third-party image requests.
The editor supports all configured locales,
summary text, tags, drafts, and
publishing.

The module does not include comments, moderation queues, scheduled publishing,
image uploads, or automatic translation. It can later integrate with the CMS
through an explicit adapter, but neither module requires the other.

## Product updates

Product Updates is a release-notes feed, separate from blog articles and CMS
pages. Each `ProductUpdate` has a locale-scoped slug, a version/release label,
a short summary, a Markdown body, draft/published status, and publication
timestamps. Administrators manage the latest 100 entries at
`/[locale]/admin/updates`; create, edit, and delete routes require the
`ADMIN` role and validate all submitted fields. Duplicate locale/slug pairs
return HTTP 409.

Public `/{locale}/updates` lists the latest 50 published entries for that
language, and `/{locale}/updates/{slug}` renders one entry. Drafts and
unpublished future entries stay private. Markdown is rendered with the shared
safe renderer; raw HTML is not enabled. Published detail pages have canonical
metadata, Article structured data, and sitemap entries. Release notes are
entered separately for each language; this module does not send newsletter or
notification messages and does not require either system.

Apply migration `20261024110000_product_updates` before deployment. The
`ProductUpdate` table and status enum are independent of `BlogPost` and
`CmsPage`; no shared content tables or integration keys are required.

## CMS page publishing

The CMS manages localized information pages independently from blog articles.
Each page is stored in its own `CmsPage` row, with a locale-scoped slug,
summary, Markdown body, draft/published state, optional SEO title/description,
directory visibility, and a no-index control. Blog posts do not share this
model or require the CMS.

Apply the `20261023120000_cms_pages` migration before deployment.
Administrators create and edit pages under `/en/admin/cms` or `/ru/admin/cms`.
The CMS-only `GET` and `POST /api/admin/cms/pages`, `PATCH
/api/admin/cms/pages/[pageId]`, and `DELETE /api/admin/cms/pages/[pageId]`
routes validate every request and require the database-backed `ADMIN` role.
Slugs are lowercase ASCII letters, digits, and hyphens and are unique within a
locale. Duplicate locale/slug pairs return HTTP 409. Publishing sets
`publishedAt`; returning a page to draft clears it. The latest 100 pages appear
in the admin list.

Published pages are served at `/[locale]/pages/[slug]`; `/[locale]/pages`
lists published pages whose directory-visibility option is enabled. Drafts and
future publication dates are never public. Each page has its own canonical URL,
Open Graph/Twitter fields, JSON-LD `WebPage` data, and optional `noindex`
directive. No-index pages are omitted from the sitemap. CMS metadata and
publishing do not depend on the SEO settings module; the Pages directory,
localized navigation, and sitemap are wired directly to CMS data.

The editor stores Markdown and renders CommonMark/GFM through the shared
`MarkdownContent` component. Raw HTML is not enabled, and Markdown image URLs
render as links without automatic third-party requests. CMS pages do not include
drag-and-drop layout editing, page revisions, scheduled publishing, image
uploads, or automatic translation.

## Customer support

Support tickets are a separate, database-backed module at
`/[locale]/dashboard/support`, with an admin queue at
`/[locale]/admin/support`. Customers create private requests with a subject,
category, and initial message; they can continue the thread from the request
detail page. Requests track open, waiting-on-customer, waiting-on-support,
resolved, and closed states, plus low/normal/high/urgent priority.

Apply the `20261023130000_support_tickets` migration before deployment.
Authenticated `GET` and `POST /api/support/tickets`, `GET` and `POST
/api/support/tickets/[ticketId]`, admin-only `GET /api/admin/support/tickets`,
and `GET` and `PATCH /api/admin/support/tickets/[ticketId]` validate requests
and scope customer reads and writes to the current user's own tickets. Requests
owned by another customer return 404. Admin routes require the database-backed
`ADMIN` role. Creating a ticket and its first message is atomic. Customer
replies reopen resolved/closed requests and mark them as waiting on support;
public admin replies mark them as waiting on the customer. Each account may
attempt at most 5 new tickets and 30 replies per fixed hourly window; attempts
are counted atomically before parsing, and customer request bodies are limited
to 64,000 bytes.

Administrators can triage status and priority and add internal notes. Internal
notes are retained in the support database and admin queue, but are excluded
from customer detail responses and account exports. Ticket threads are plain
text, not HTML. The account JSON export includes a customer's ticket history
and public messages. Ticket views are limited to the latest 100 tickets and
latest 200 messages per thread.

The support module uses no email, notification, storage, or external help-desk
provider; customers and staff communicate through the authenticated site.
Attachments, email alerts, assignment, and SLA automation are not included.

## Community forum

The public community at `/[locale]/community` is a global discussion space,
not a support queue or organization workspace. Visitors can read open and
locked topics; signed-in members can create questions, ideas, discussions, and
showcases and reply to open topics. Topics and replies are stored separately
from support tickets. Each topic is limited to 5 creations per account per
hour, and replies to 20 per account per hour. Content is validated as bounded
plain text/Markdown and rendered with GFM without enabling raw HTML. Reports
are also limited to 10 per account per hour. Topic, reply and report quotas use
atomic fixed hourly windows starting with the first attempt, including invalid
attempts; concurrent requests cannot bypass them. Topic/reply bodies are limited
to 64,000 bytes, and report bodies to 4096 bytes.
Reply creation serializes with topic moderation: if locking or hiding the
topic commits first, the pending reply is rejected rather than added afterward.

Apply the `20261024180000_community_forum` migration before deployment.
`GET`/`POST /api/community/topics`, `GET
/api/community/topics/[topicId]`, `POST
/api/community/topics/[topicId]/replies`, and `POST /api/community/reports`
provide the public API. Reporting requires authentication; a member can report
each topic or reply once, and cannot report their own content. Hidden content
is excluded from public queries and the sitemap. The index returns the latest
50 topics and a thread shows up to its latest 100 replies; cursor pagination
and full-text search are future extensions.

Administrators use `/[locale]/admin/community` to review pending reports,
resolve a report while hiding or retaining its content, dismiss a report, pin
topics, lock/reopen replies, hide a topic, or restore previously hidden topics
and replies. `PATCH /api/admin/community/reports/[reportId]`,
`PATCH /api/admin/community/topics/[topicId]`, and
`PATCH /api/admin/community/replies/[replyId]` enforce the database `ADMIN`
role independently of the page. Reporter identity and report reasons
are only returned to administrators. Deleting a user cascades their topics,
replies, and reports; the account export includes the user's forum data. The
public discussion pages are included in the sitemap.

A basic forum is moderate in complexity: the core needs topics, replies,
authorization, bounded listing, localization, and safe rendering. Operating a
large public community is a separate, higher-effort phase. Useful follow-ons
include reactions and accepted answers, user mentions, notifications,
anti-spam reputation, attachments, community guidelines, moderator roles,
appeal workflows, and search. They are intentionally not coupled to Support,
Notifications, Email, or organization tenancy in this first implementation.

## Curated directory

The directory is a localized catalog of team-curated resources, tools, or
services, separate from CMS pages, support tickets, and user accounts. Each
`DirectoryEntry` has a locale-scoped slug, name, summary, description,
administrator-defined category, tags, optional HTTPS website URL, featured
flag, draft/published state, and a no-index control. It does not expose user
profiles or accept public submissions.

Apply the `20261023140000_directory_entries` migration before deployment.
Administrators manage entries under `/en/admin/directory` or
`/ru/admin/directory`; admin-only `GET` and `POST
/api/admin/directory/entries`, `PATCH
/api/admin/directory/entries/[entryId]`, and `DELETE
/api/admin/directory/entries/[entryId]` validate changes. Slugs are lowercase
ASCII letters, digits, and hyphens and are unique per locale. External website
links must use HTTPS and cannot contain URL credentials. Publishing sets
`publishedAt`; returning an entry to draft clears it. Duplicate locale/slug
pairs return HTTP 409.

The public index at `/[locale]/directory` filters published entries by text
and category, places featured entries first, and shows up to 60 results.
Published details are at `/[locale]/directory/[slug]`. Drafts and future-dated
entries are not visible. Canonical metadata and WebPage structured data are
generated for each detail page; no-index entries remain accessible but are
excluded from the sitemap. The directory can operate without CMS, Support,
Marketing, or user-profile data. It does not include public submissions,
reviews, paid placements, images, or submission moderation.

## Notifications

The authenticated dashboard includes a persistent inbox and account settings
for three independent delivery channels: email to the account address, Web
Push to registered browsers, and a user-configured webhook. The inbox works
with all external channels disabled. Stripe subscription-status changes,
successful payments, and failed subscription invoices create idempotent,
localized notifications in the same database transaction as their Stripe
event. The newest 50 appear in the inbox; the account export includes the
complete notification history and read state.

Each opted-in channel gets a durable delivery job in the same transaction as
the notification. A crash before dispatch therefore cannot lose the job.
Delivery status and attempt count are recorded per notification/channel;
one failure does not block the inbox or other channels. Email uses the
configured Resend, SendGrid, or SMTP adapter. Push uses browser service workers and VAPID and requires
`NEXT_PUBLIC_VAPID_PUBLIC_KEY` plus server-only `VAPID_PRIVATE_KEY` and
`VAPID_SUBJECT`; clients without the configured keys cannot subscribe. The
service worker is `public/sw.js`.

Webhook URLs must use HTTPS on the standard port and resolve only to public
IPv4 addresses. DNS is checked at configuration and delivery time, the selected
public address is pinned for the request to prevent DNS rebinding, and
redirects are not followed. The generated HMAC secret is encrypted at rest
using a key derived from `AUTH_SECRET`, returned only when created or rotated,
and never included in the data export. Each request includes
`X-Launchpad-Timestamp` and `X-Launchpad-Signature: sha256=...`; the HMAC-SHA256
input is `<timestamp>.<raw request body>`. Webhook consumers should verify the
signature against the unmodified request body and reject timestamps older than
five minutes. Rotating `AUTH_SECRET` requires rotating webhook secrets.
Rotating VAPID keys also requires users to disable and re-enable browser push
subscriptions. Channel deliveries are claimed separately so concurrent Stripe
retries do not normally send duplicates; webhook consumers should still
deduplicate using the notification `eventKey`. Ownership tokens fence stale
workers; expired five-minute leases can be reclaimed. Delivery remains
at-least-once: a crash after provider acceptance but before the database commit
can cause a duplicate, especially with SMTP or SendGrid.

For automatic recovery, set a unique 32-character-or-longer
`NOTIFICATION_CRON_SECRET` and configure an **external scheduler** to call
`GET /api/cron/notifications` every minute with
`Authorization: Bearer <NOTIFICATION_CRON_SECRET>`. This endpoint is not in the
default Vercel cron configuration; without this scheduler, only immediate
delivery and user-triggered retries run. Each invocation examines at most ten
jobs with concurrency five and a sixteen-second external network budget.
Failed jobs wait 1, 2, 4, and 8 minutes before the next attempt; after five
attempts they remain visibly failed until an explicit user retry. The user's
current channel settings are checked on every attempt; an opted-out channel
becomes `SKIPPED`, not successfully delivered. Manual retries are limited to
ten per account per hour and never reclaim a live lease.

Deploy `20261022134000_notification_outbox` with `npm run db:deploy`.
Pause dispatchers and drain old workers before deploying the migration and
new worker code. Existing delivery rows are retained and recoverable according
to their attempt budget; historical inbox entries without delivery rows are
not retroactively sent, since their original opt-in state is unknown.

The `NotificationPreference`, `Notification`, `NotificationDelivery`, and
`PushSubscription` models are added by the
`20261018020000_notifications` migration. Apply it with `npm run db:deploy`.

## Newsletter and admin campaigns

The footer newsletter form requires an explicit consent checkbox. The
`NewsletterSubscriber` record stores the normalized address, subscription
time, and a private unsubscribe key. Re-subscribing reactivates that address.
An email's `List-Unsubscribe` headers support one-click POST unsubscription;
opening the link with GET shows a confirmation page and does not silently
unsubscribe the recipient. The confirmation form body is limited to 4096 bytes.

Admins can create plain-text campaigns at `/{locale}/admin/email`. Creating a
campaign snapshots the currently subscribed audience and the unsubscribe key
for each recipient with one server-side SQL INSERT/SELECT, without loading
subscriber addresses into application memory. Draft and audience commit together;
the statement has a 15-second timeout and failures roll back the draft. This
statement's database snapshot excludes later signups; consent is still checked
again before sending. The admin then sends the campaign in resumable batches of
10, with at most five concurrent provider requests. Each recipient is checked
against the current subscription before sending, so an address that opted out
after the snapshot is suppressed. Delivery status is recorded per recipient;
failed sends can be retried, and stale claims are reclaimed after 15 minutes.
Resend idempotency keys make retries safe if a send succeeded but the
application did not record the response. SendGrid receives the key as tracing
metadata only, so a retry after an uncertain response can duplicate a message.

Campaign sending requires a configured `EMAIL_PROVIDER`, its matching API key,
and a verified `EMAIL_FROM` sender.
This is a basic consent-based broadcast tool; it does not include a visual
editor, segmentation, scheduling, double opt-in, or compliance/legal advice.

The `NewsletterSubscriber`, `EmailCampaign`, `EmailCampaignRecipient`, and
`TransactionalEmail` models are introduced by the
`20261018000000_email_campaigns` migration. Apply committed migrations with
`npm run db:deploy` in shared/production databases. For a disposable local
database, `npm run db:push` can synchronize the schema.

## Newsletter marketing automation

The admin Marketing page at `/{locale}/admin/marketing` can create a draft
sequence of up to 10 plain-text email steps. Each step has a delay from 0 to
8760 hours: the first delay starts at signup, and later delays start after the
previous email is accepted by the selected provider. Activating a sequence enrolls future
newsletter signups only. The signup API requires explicit consent and creates
the enrollment in the same transaction as the subscriber record; existing
subscribers, waitlist entries, anonymous attribution events, and transactional
emails are not enrolled.

Each message uses the shared email-provider newsletter transport, `List-Unsubscribe`
headers, and the subscriber's private unsubscribe link. Delivery checks the
current subscription before each message. Pausing a sequence stops new
deliveries without deleting pending enrollments. The dispatcher processes at
most 10 due enrollments per request, up to five concurrently; transient
failures retry after five minutes, up to five attempts. Stale claims are
recovered after 15 minutes. Resend guarantees provider-side idempotency;
SendGrid tracing metadata does not prevent duplicate delivery after an
uncertain response.

Set `MARKETING_CRON_SECRET` to a unique value of at least 32 characters and
configure an external scheduler to call `GET /api/cron/marketing` with
`Authorization: Bearer <MARKETING_CRON_SECRET>` (for example, every minute).
Without the secret or scheduler, drafts and subscriber signups still work, but
scheduled emails are not dispatched. The selected email provider must also be
configured before a sequence can be activated. Apply migration
`20261024160000_marketing_automations` after the email migration. This module
does not add subscribers to the separate waitlist, use behavioral/checkout
triggers, or send push/webhook messages; those require separate consent and
integrations.

## Dashboard and analytics

The customer dashboard shows account status, creation date, and the latest
subscription status; the billing screen shows recent payments. The protected
API card is an example integration point, not product analytics.

Admin analytics are available at `/{locale}/admin/analytics` to administrators.
The 7-, 30-, and 90-day ranges control new-user counts, daily user registrations,
successful-payment counts, and daily successful payments. Payment counts
include paid and no-payment-required records; revenue sums only `paid` records
and is grouped by currency to avoid combining unlike currencies. Subscription
status totals are all-time. Daily buckets use UTC dates.

Universal product analytics are available in the same admin report. Visitors
must opt in from dashboard settings before the app records page views or
events. Product code can add events with
`trackProductEvent("feature_used", { feature: "reports" })` from
`src/lib/product-analytics.ts`. Events are deduplicated by a random event key
and grouped by event name, page path, and a server-issued browser session ID.
Apply migration `20261022150000_analytics_sessions` before deployment. Sessions
use a signed, HttpOnly, SameSite=Lax cookie scoped to `/api/analytics`, Secure
on HTTPS deployments, and expire after 24 hours. Browser tabs share this cookie.
Client-supplied legacy `sessionId` fields are accepted but ignored; historical
events without a server session are never retroactively claimed. Before
authentication, events remain anonymous. After sign-in, the client calls
`POST /api/analytics/identify`; the server derives the account from authentication
and binds only the signed browser session, atomically with its anonymous events.
Concurrent claims cannot assign one session to different accounts. Logout and
account changes rotate the session on the next analytics request. Client deliveries
are serialized, bounded to 20 queued requests, and time out after 8 seconds.
The retention cron also deletes expired session records in bounded batches;
repeat when `hasMore` is true. The endpoint rejects client-supplied `userId`; event capture also derives
the account from the authenticated session. Account-linked analytics events
are included in the account's JSON export. Deleting an account sets these
event references to null so aggregate events remain until their configured
retention expires. The API
rejects query strings, URL/email-shaped values, identifying property names,
oversized payloads, and excessive global/account event rates. Never send names,
contact details, credentials, free-form user content, or other personal data
in event properties.

The admin report shows event totals, browser sessions, distinct identified
accounts, daily event volume, common event names, and page paths for the
selected 7-, 30-, or 90-day period. Identified account activity is stitched
across that account's browser sessions; anonymous sessions remain separate.
Funnels use this same identity rule. Cohorts still group browser sessions and
do not measure account-specific retention. Marketing attribution remains a
separate, campaign-scoped module and works independently. Analytics consent is
local to the current browser and defaults off. Disabling it stops future
collection; the retention job eventually removes the underlying events.

The admin report also includes an ordered funnel builder. Enter 2–6 event
names; an identified account advances only when each next event occurs later
than the previous step for that account, including across its sessions. An
anonymous visitor advances within the same browser-tab session. Each step
reports identified-account and anonymous-session counts separately, plus
conversion from the first step and step-to-step drop-off. Event names must
follow the product event naming rules.

The cohort builder lets an administrator choose the entry event and return
event. It groups browser sessions by the week of their first matching entry
event within the selected date range; period columns show the share with the
chosen return event in each completed week. Incomplete weeks are marked
unavailable instead of being shown as zero retention. Cohorts remain
session-based and do not measure an account-specific retention curve.

Product event retention defaults to 365 days and can be configured from 30 to
2,555 days with `PRODUCT_ANALYTICS_RETENTION_DAYS`. Set a separate random
32-character-or-longer secret as both Vercel `CRON_SECRET` and
`ANALYTICS_CRON_SECRET`. The included `vercel.json` invokes
`GET /api/cron/analytics-retention` daily at 03:00 UTC. On other hosts, schedule
that endpoint daily with its bearer token. The endpoint
deletes expired product events in batches of 10,000, with a maximum of 10
batches (100,000 rows) per invocation; schedule another run when the response
reports `hasMore`. Without the secret and scheduler the cleanup does not run.
The job deletes only `ProductAnalyticsEvent` rows; it does not delete
marketing-attribution records, user accounts, billing records, or other data.
Apply migration `20261024120000_product_analytics` and then
`20261024220000_analytics_identity` before deployment.

## Localization, themes, and UI

Locale routes are discovered from canonical JSON catalog filenames in
`messages/`; English (`en.json`) is the required fallback catalog. The current
project includes catalogs for English, Arabic, Bengali, German, Spanish,
French, Hindi, Indonesian, Italian, Japanese, Korean, Dutch, Polish,
Portuguese, Russian, Turkish, Vietnamese, and Chinese. To add a locale, add a canonical BCP-47 filename such as
`messages/fr-CA.json` and group its messages by module, retaining the existing
namespace names inside each module. `npm run i18n:sync` validates the catalogs
and generates `src/i18n/locales.generated.ts`; development, build, lint,
type-check, and i18n-test commands run this sync automatically. Existing
`useTranslations` namespaces remain available, catalog keys are checked for
parity, incomplete catalogs inherit missing messages from English, and the locale picker, profile settings,
content editors, validation, metadata, alternates, and sitemap all use the
generated locale list. Product-builder copy added before a locale exists
inherits English text for that locale. Apply migration
`20261024210000_i18n_locale_tags` before storing locale tags beyond the existing
short codes; database locale columns then accommodate full BCP-47 tags. Text
direction is derived from the locale's likely writing script, so RTL locales
automatically set `dir="rtl"`.

The settings page includes a theme editor with five accent presets, a custom
color input, minimum 4.5:1 contrast
validation for primary button text, three global corner-radius values, and
light/dark/system mode selection. Color, radius, and mode changes preview
immediately. Accent and radius are stored in browser local storage; the mode is
managed by `next-themes`. Settings apply to the current browser, are not
synchronized to the account, and can be reset from the editor. The shared
Tailwind/shadcn-compatible components, layout, and responsive navigation are
intended to be extended with the product.

The same settings area also includes a UI/UX editor that stores browser-local
layout preferences: comfortable or compact density, standard or wide content
width, sticky or static top navigation, reduced-transparency mode for header
surfaces, and standard or strong focus-ring visibility. A dedicated provider
applies these values through CSS variables and `data-ui-*` attributes. Controls
that rely on the design system (buttons, inputs, layout containers, and header
surfaces) adapt immediately; other modules remain functional if UI/UX settings
are reset or unavailable.

The animation editor in dashboard settings independently stores browser-local
motion preferences: enable/disable motion, entrance effect (fade-up, fade,
scale, or none), duration (100–1000 ms), easing curve, travel distance
(0–32 px), stagger delay (0–300 ms), hover motion, and whether to honor the
operating system's reduced-motion preference. A site-wide provider applies the
validated values as CSS custom properties after load and synchronizes changes
between tabs. Landing feature cards use the configurable entrance and hover
effects; shared Tailwind transitions use the motion duration token. Reduced
motion is respected by default, and disabling motion also reduces global
animation and transition durations. Preferences are per browser and do not
require the theme editor or account storage.

## Product builder

Administrators can configure the shared product experience at
`/{locale}/admin/product-builder`. Apply migration
`20261024130000_product_builder` before deployment. The editor stores validated
draft and published JSON configurations in a singleton database record; saving
a draft does not change the live product, while publishing updates it for all
configured locales. When a new locale is added, missing product-builder copy
uses English until an administrator supplies a translation.

The landing-page editor changes localized hero and feature-card copy, trust
labels, button destinations, section visibility, and section/card order. CTA
destinations are restricted to the workspace dashboard and pricing page rather
than accepting arbitrary URLs. The workspace editor reorders and hides known
navigation links and the signed-in overview's built-in widgets (onboarding,
account and plan summaries, trusted plugin slots, and the protected API
example); the overview link remains visible, and the AI link is also gated by
AI provider configuration. Hiding a link or widget only changes presentation:
it does not disable its service, permissions, API endpoints, or direct routes.

Administrators can also publish the shared primary brand color and corner
radius. The brand is applied at the document root and adapts to light/dark mode;
the foreground color is selected automatically for contrast. A user's
browser-local theme editor can override this shared default. Existing stored
configurations without brand settings receive the starter defaults when parsed.

This builder composes the starter's existing sections and navigation registry;
it does not execute uploaded code, create arbitrary components, replace CMS
pages, change application routes/business logic, or edit browser-local UI/UX
and motion preferences. The section and navigation registries are deliberately
allow-listed and validated on the server.

## Settings, notifications, marketing, and SEO

The account settings page edits the display name and stores a preferred
language from the configured 18 locale catalogs
plus an IANA time zone. Account creation and payment dates use that time zone
in the customer dashboard. The settings page also links to
an authenticated JSON export of the account profile, Stripe subscriptions,
store-purchase summaries, payment history, stored-file metadata, and newsletter subscription status;
it intentionally excludes OAuth tokens, sessions, unsubscribe secrets, webhook
signing secrets, Google purchase tokens, and browser push credentials. Notification preferences and inbox history are also
included. The JSON download streams with backpressure and 100-row keyset pages,
including separate pages for nested AI/support messages; it never loads full
histories into the server heap. Cursor IDs remain internal, and internal support
notes remain excluded. Cancellation stops further page reads. Collections use
deterministic ID order rather than timestamp order, with a creation-time cutoff
captured at export start; updates/deletions during the download may still appear.
This is not a database-wide snapshot or a background archive job. The endpoint
allows two attempts/account/hour and requests a 300-second runtime budget; configure
your hosting/proxy limits accordingly. Failures during streaming abort the download
rather than returning a successfully closed, truncated JSON file. Billing settings and plan management are handled separately by the
billing page and Stripe Customer Portal. Sonner provides short-lived
success/error toasts in addition to the persistent notification inbox.

### Private object storage

File storage is an optional dashboard-settings module. Configure
`S3_REGION`, `S3_BUCKET`, `S3_ACCESS_KEY_ID`, and `S3_SECRET_ACCESS_KEY` to
enable it; leave them unset to keep the module unavailable without affecting
other features. `S3_ENDPOINT` supports S3-compatible providers such as AWS S3,
Cloudflare R2, and MinIO, and `S3_FORCE_PATH_STYLE=true` is available for
providers that require path-style addressing. Remote endpoints must use HTTPS;
plain HTTP is accepted only for loopback development endpoints. Credentials stay server-side.
Apply migration `20261020000000_object_storage` before enabling storage.

Keep the bucket private and block public access. Configure its CORS policy to
allow browser `POST` requests from the application origin for direct uploads.
The server creates a random per-user temporary object key under `pending/` and a five-minute presigned
POST policy; it accepts JPEG, PNG, WebP, PDF, and plain text files up to 10 MiB,
with a maximum of 100 database records per account. After upload, the server
checks the object size and content type, copies with an ETag precondition to a
new `ready/` key, and checks the copy before atomically publishing it. Replaying
the upload policy cannot replace an accepted file. Copy attempts are durably
tracked for cleanup even if finalization crashes. Downloads
use five-minute presigned GET URLs and all file operations verify ownership.
The POST policy allows a small multipart-form overhead above 10 MiB; completion
still requires an exact match to the declared (10 MiB maximum) file size.
Abandoned pending records older than 15 minutes are removed on the next file
list or upload request. Cleanup atomically claims `DELETING` before touching S3;
completion cannot publish a claimed file. Failures retain retryable records.
Apply migration `20261022152000_storage_finalization`. The storage credentials
must support server-side CopyObject; bucket versioning is not required.
Legacy READY objects are copied to protected keys before issuing new downloads.
When upgrading, pause old upload/completion traffic and wait five minutes for
previously signed upload/download URLs to expire.
Set a separate `STORAGE_CRON_SECRET` (at least 32 characters) and externally
schedule `GET /api/cron/storage` with `Authorization: Bearer <secret>` every
minute for automatic recovery without user traffic. Each call handles at most
two abandoned/deleting files and two temporary/orphan keys; repeat on `hasMore`.
S3 HEAD/copy/delete operations have eight-second deadlines and no implicit retries.
Configure a bucket lifecycle rule to expire `pending/` objects after one day as
an additional safeguard against upload-policy replays after metadata deletion.
Deleting an account cascades its metadata, but does not delete remote objects;
configure a cleanup process for account deletion. The module does not scan files for malware, so
it rejects active-content formats such as SVG and HTML. User data exports
include file metadata, never object credentials or URLs.

## Referral tracking and partner rewards

Apply migration `20261021000000_referral_tracking` before enabling the partner
pages or attribution routes.

Authenticated users can apply to become partners. Applications start in
`PENDING`; an administrator activates or pauses partners and configures a
commission rate (20% by default, configurable from 0% to 100%). Active partners
share `/r/{code}` links. A valid link records a timestamped click without an IP
address or user-agent, issues a signed, HTTP-only, SameSite=Lax cookie for 30
days, and redirects to the localized home page. The first active referral
click is retained; after sign-in, one account can be attributed to at most one
partner. When the hosting proxy provides a client address, the endpoint is
rate-limited by its keyed hash; raw addresses are not stored. This starter has
no consent manager; deployments subject to consent requirements must disclose
and gate the 30-day attribution cookie accordingly. Self-referrals are rejected.

The referral dashboard shows visits, referred accounts, and reward totals by
currency and state. Verified successful Stripe one-time payments and paid
subscription invoices accrue an idempotent reward using the partner's rate at
the time of payment. Amounts are stored in the payment's minor currency unit
alongside the source amount and rate. Rewards begin as `PENDING`; admins can
approve or void them, then mark approved amounts paid. Each administrative
change is recorded in the security audit log. This is accounting, not a payment
processor: payouts are made outside the app. Apply migration
`20261022132000_payment_reversals`. Approval and payout have a 30-day hold.
Subscribe to `charge.refunded` and `charge.dispute.*` events as well as the
existing Checkout/invoice events. Reconciliation reads current Stripe state
under transaction locks; partial refunds retain access and reduce commissions
proportionally. Full refunds and open/lost disputes revoke the affected lifetime
purchase, but not another valid lifetime payment. Winning a dispute restores
that purchase. Subscriptions retain their Stripe billing dates and renewal
settings; `Subscription.paymentBlocked` suspends the current period when all
payments covering it have been refunded/disputed. Authorization for paid
features must check this flag as well as provider status and period end.
Previously paid cash is retained in `ReferralReward.paidAmount`; commission
corrections create a per-currency carry-over balance that automatically adjusts
the next recorded cash payout. The admin must verify the displayed cash amount
before recording an external payout; a changed balance returns 409.
Unknown payments fail with a retryable webhook error, not a successful ignore.
Existing Checkout payments are linked lazily by session ID. Before upgrading an
installation with historical billing, pause Checkout/webhook processing and
run `npm run billing:backfill` to verify historical sources without writing,
then `npm run billing:backfill -- --apply` to backfill invoice PaymentIntent IDs,
subscription IDs/coverage dates and lifetime payment plans from Stripe; old
unlinked records cannot be safely reconciled from an individual webhook alone.
The command links all sources first, then reconciles current provider state;
it is bounded to 50 records per page and can be rerun after failure. Resume billing
only after it succeeds. This integration
supports a single PaymentIntent per invoice; split invoice payments require an
explicit allocation implementation and are rejected rather than guessed.
`npm run test:billing:integration` runs real PostgreSQL reversal tests when
`BILLING_TEST_DATABASE_URL` points to the disposable localhost `/ci` database;
CI migrates a dedicated PostgreSQL service before running them.
The module does not create commissions for App Store/Google Play purchases because those
purchase records do not contain a trusted price. Referral tracking works
without the optional Marketing analytics module; payment commissions require
the Stripe billing integration.

The localized landing and pricing pages are the marketing surface. The admin
Marketing section provides a UTM URL builder for those public destinations and
campaign-attributed reporting for page views, CTA clicks, successful newsletter
signups, and checkout intents. A tagged visit establishes tab-scoped attribution
in `sessionStorage`; campaign events contain only the UTM fields, event type,
public page path, random idempotency key, and timestamp. No visitor ID, email,
IP address, or referrer is stored. Only visits carrying both `utm_source` and
`utm_campaign` are measured, and administrator browsing is excluded. Reports
cover 7, 30, or 90 days. Apply migration
`20261019000000_marketing_toolkit` before enabling the admin report or event
capture route. The existing newsletter campaign tool remains separate and
continues to work without Marketing tracking. Newsletter automation is a
separate consent-based email workflow and does not use anonymous event
attribution to identify subscribers.

## Advertising account and campaign management

Apply migration `20261024190000_ads_management` and open
`/{locale}/admin/ads` as an administrator. Register account identifiers and
their reporting currency, create local campaign records, and import daily
network-exported CSV reports. Supported platform labels include Google Ads,
Meta Ads, LinkedIn Ads, TikTok Ads, Microsoft Ads, and Other; they are
organizational labels, not authenticated provider connections.

CSV imports are limited to 2 MB and 5,000 data rows. Common Google Ads and Meta
column labels are recognized for date, campaign, impressions, clicks, spend,
conversions, and revenue. Dates may use `YYYY-MM-DD` or `M/D/YYYY`. Every row
must match a campaign name in the selected account (case-insensitive), and
repeated rows for a campaign/day are aggregated before an upsert. Campaign names
are unique within an account regardless of case; rows that would match several
existing legacy campaigns are rejected with 409 instead of being assigned to
one of them. Re-importing
the same campaign/day replaces its prior values rather than double-counting.
Metrics are interpreted in the account's configured ISO currency; there is no
currency conversion or independent verification of the uploaded report.

Campaigns store a local landing path, UTM campaign key, objective, optional
daily budget, and tracking status. The generated link uses the existing
marketing attribution helper (`utm_medium=paid` and a platform-specific
`utm_source`) and campaign analytics can be compared with those first-party
events. Campaign/account status and budget are informational only: changing
them does not pause an external campaign or affect an advertising account.
OAuth credentials are not collected or stored, and live network APIs, ad
creation, and automatic synchronization are not implemented. The Ads records
and imported metrics work without the Marketing module; the attributed-event
comparison uses Marketing data when available.

Landing feature cards use the configurable motion editor; the mobile site
navigation collapses into a menu. Global motion and visual tokens are CSS
variables in `src/app/globals.css`. There is not yet a per-element timeline
authoring tool.

## Search optimization (SEO)

The admin SEO section at `/{locale}/admin/seo` edits the title, description,
keywords, optional social-preview image, and indexability of the localized home
and pricing pages. Changes are validated on the server and stored by page and
locale in `SeoMetadata`; unedited entries use the defaults from the message
catalogs. Titles are limited to 70 characters and descriptions to 320
characters. Keywords are included in metadata for consumers that use them, but
major search engines may ignore the keywords meta tag.

The same admin page runs an on-demand technical audit over localized public
listing pages and currently published CMS pages, blog posts, product updates,
directory entries, and community topics. It counts indexable and explicitly
excluded content, and recommends fixing missing or duplicate titles,
titles outside 10–60 characters, and descriptions outside 50–160 characters.
These lengths are editorial heuristics, not search-engine guarantees. The
audit reads the application's own database and translation catalogs; it does
not crawl the deployed site, validate live HTTP responses or structured data,
or report impressions, rankings, clicks, or indexing status.

For these public pages, `src/lib/seo.ts` generates canonical URLs and `en`,
`ru`, and `x-default` language alternates from `NEXT_PUBLIC_APP_URL`. Open
Graph and Twitter cards share the configured title and description and use the
optional HTTPS or same-origin image URL. Setting no-index adds `noindex,
nofollow` directives and removes that locale/page from the generated
`/sitemap.xml`.

`/robots.txt` points crawlers to the sitemap and disallows API, admin,
dashboard, sign-in, and verification-request routes. Admin, dashboard, login,
and verification-request pages also emit `noindex`. The home page emits
Organization and WebSite JSON-LD; pricing emits WebPage JSON-LD. The localized
root layout sets the metadata base, Open Graph locale, and optional Google/Bing
ownership verification tags. Configure `NEXT_PUBLIC_APP_URL` to the actual
canonical HTTPS origin and optionally set
`NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION` or
`NEXT_PUBLIC_BING_SITE_VERIFICATION` from the provider's webmaster console.

The generated `/sitemap.xml` includes localized listing pages, including the
blog index, and currently published content. CMS and directory records marked
no-index are left out. Apply `20261018030000_seo` with `npm run db:deploy`
before deploying the admin editor or serving the dynamic metadata and sitemap
routes. This toolkit does not provide search-traffic analytics, a redirect
manager, or automatic keyword research; the CMS manages localized Markdown
pages separately from the blog. Search engines must crawl and index the
deployed site before changes affect results.

## Integration catalog

Administrators can review provider setup at `/{locale}/admin/integrations`.
The catalog covers Google and GitHub OAuth, Stripe, Resend/SendGrid/SMTP, S3-compatible
storage, Web Push, App Store and Google Play purchases, an OpenAI-compatible AI
provider, and Google/Bing site ownership verification. It reports
`Configured`, `Incomplete`, or `Not configured` by checking only whether the
required environment variable names have non-empty values. The UI shows
missing variable names and links to provider setup and related product
features; it never returns or displays credential values.

The catalog does not test credentials against provider APIs or create provider
accounts. Configure secrets in the hosting platform's server-side environment,
then redeploy where required. OAuth redirects and token exchange are
provider-specific: some services support them, while Stripe platform keys,
email API keys, storage credentials, Apple certificates, and Google Play
service-account credentials require separate provider setup. A configured
status confirms presence only, not that credentials are valid or the provider
is reachable.

## Security boundaries and operational scope

The starter validates environment variables at runtime, validates profile,
admin, newsletter, and checkout input with Zod, checks user/admin identity on
the server, validates Stripe webhook signatures, and uses transactional
idempotency for Stripe events and billing-email delivery. Subscription
entitlements are derived from verified Stripe events rather than return URLs.
Newsletter unsubscribe tokens are private, and campaign sends re-check consent.
Notification webhooks use HTTPS, reject non-public IP destinations, pin DNS
results for each request, and sign the raw payload with a timestamp. Signing
secrets are encrypted at rest, and notification APIs restrict access to the
owning account.

The Security module adds database-backed fixed-window limits of three magic
link sends per normalized email and 30 per request IP in 15 minutes. The IP
limit uses proxy headers, so a self-hosted deployment must configure its trusted
proxy to overwrite those headers. Identifiers are HMAC-SHA256 hashed with
`AUTH_SECRET` before storage; old rate buckets are pruned during magic-link
requests. The user's `sessionVersion` is checked by
the JWT callback against the database. Users can revoke all account sessions,
and role/status changes for other accounts also increment that version so
existing tokens stop working. A security audit event is written for successful
sign-ins, session revocations, admin role/status changes, and notification
webhook-secret rotations. Admins can review the latest 100 events at
`/{locale}/admin/security`; the audit data does not retain client IP addresses.

Global response headers include `X-Content-Type-Options`, `X-Frame-Options`,
`Referrer-Policy`, and a restrictive `Permissions-Policy`. Production responses
also receive HSTS. A strict Content Security Policy is not enabled because the
application currently uses inline style attributes and provider integrations;
add a nonce-based CSP after auditing those surfaces.

These protections are a baseline, not a complete security program. The starter
includes optional TOTP MFA with one-time recovery codes, but does not include a
password-based account recovery workflow, an external WAF, or automated
vulnerability monitoring. Rate limiting and audit logging require the
`20261019010000_security_controls` migration. Configure additional controls
according to the deployed product's threat model.
GitHub Actions CI validates pull requests and pushes to `main`; Vercel Git
integration provides Preview/Production deployments when connected and
configured. Store verification depends on correctly
configured Apple/Google credentials and production store dashboards; native
examples are integration samples, not complete applications.

## Deployment and database lifecycle

The CI workflow uses lockfile-based installs, Prisma generation/validation,
type-checking, formatting checks, linting, focused utility tests, and a
production build without production credentials or a live database. Require
its `CI / validate` check on protected `main`; connect Vercel Git integration
for Preview and Production deployment. CI never receives production database
credentials.

The standard `npm run build` script runs `prisma migrate deploy` before the
application build only when `VERCEL_ENV=production`. Vercel's default Next.js
build invokes this package script, so no custom Build Command is needed. Local
and Preview builds skip production migrations. Set both `DATABASE_URL` and
`DIRECT_URL` in the Vercel Production environment, with `DIRECT_URL` pointing
to a direct PostgreSQL connection suitable for migrations. Keep Preview on an
isolated database and apply migrations there separately when validating schema
changes. If you replace Vercel's default build command, retain `npm run build`
or provide an equivalent guarded migration step; `npm run build:app` is the
internal application-only build.

Review migrations before merging and use backward-compatible expand/contract
changes because the previous app version may still be serving during rollout.
A migration failure blocks the new build; an already-applied migration is not
automatically rolled back. For non-Vercel hosting, run `npm run db:deploy` in
the release pipeline before starting the application version that requires it.

Local development can use the PostgreSQL service in `docker-compose.yml`.
Production deployments need PostgreSQL, `DATABASE_URL`, `DIRECT_URL`, and
`AUTH_SECRET`. Configure `NEXT_PUBLIC_APP_URL` to the canonical HTTPS origin
for unsubscribe links; configure `AUTH_URL` for Auth.js callbacks. Configure
selected email-provider and Stripe secrets only for the integrations being used. See the
deployment instructions in the root README before applying migrations or
switching Stripe from test to live mode.
