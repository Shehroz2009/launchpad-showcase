# Vercel deployment checklist

Launchpad is a standard Next.js application that can be deployed to Vercel.
Vercel does not provision the external services used by your selected modules.
Configure the application, database, secrets, and provider accounts separately.

## Before deployment

1. Run `npm run setup` from a clean project archive or checkout and select the
   modules you intend to deploy.
2. Create a Vercel project connected to the configured Git repository.
3. Provision PostgreSQL. Configure a pooled runtime connection for
   `DATABASE_URL` when required by your database provider and a direct
   migration connection for `DIRECT_URL`.
4. Set required application variables in the correct Vercel environments,
   including the database URLs, strong `AUTH_SECRET`, and public application
   URL.
5. Add provider credentials only for enabled features. Use test credentials
   for Preview where supported and production credentials only in Production.
6. Set `NEXT_PUBLIC_APP_URL` and Auth.js URL/trusted-host settings to the
   deployed HTTPS origin.

Use the generated `.env` and `.env.example` as the source of truth for exact
variable names. Do not place server secrets in `NEXT_PUBLIC_*` variables.

## Database migrations

The production build applies committed Prisma migrations through the project's
Vercel build flow. Ensure `DIRECT_URL` can reach the production database during
the build and that the deployment role has the required migration privileges.
Back up important production data and review pending migrations before rollout.

## Provider configuration

- Register HTTPS OAuth callback URLs for each enabled identity provider.
- Configure the email provider's sender domain and credentials, then verify
  delivery and sign-in links.
- Register the Stripe webhook endpoint at
  `https://your-domain.example/api/stripe/webhook` and configure its signing
  secret, API key, and product price IDs.
- Configure storage, AI, push, Apple App Store, and Google Play credentials
  only if those modules are selected.
- Configure any provider-side lifecycle notifications to target the deployed
  webhook endpoints required by the corresponding module.

## Verify the deployment

1. Check deployment logs and database migration results.
2. Test sign-in, account access, and admin authorization.
3. Verify email delivery, OAuth callbacks, and user-facing error handling.
4. Test Stripe checkout, Customer Portal, and webhook delivery using the
   appropriate test/live configuration.
5. Confirm retries and duplicate webhook events do not apply changes twice.
6. Check responsive public pages, localized routes, consent, and SEO metadata.

Vercel Git integration can build Preview and Production deployments. The setup
wizard does not create the Vercel project, provision cloud resources, or
configure provider accounts for you.

For setup, module limits, and security boundaries, see
[QUICKSTART.md](./QUICKSTART.md) and
[FEATURE-GUIDE.md](./FEATURE-GUIDE.md).
