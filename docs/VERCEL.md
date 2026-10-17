# Vercel deployment checklist

The full source kit is a standard Next.js application intended for deployment
to Vercel or another compatible host. This public showcase repository does not
contain deployable application code.

1. Provision PostgreSQL. Use a pooled runtime URL for `DATABASE_URL` when
   required by your provider and a direct URL for `DIRECT_URL`.
2. Import the purchased source repository into Vercel.
3. Add the required variables to the correct Vercel environments:
   `DATABASE_URL`, `DIRECT_URL`, `AUTH_SECRET`, `AUTH_URL`, and
   `NEXT_PUBLIC_APP_URL`.
4. Add optional email, OAuth, and Stripe variables only when enabling those
   integrations. Use separate test credentials for Preview and live
   credentials for Production.
5. Apply Prisma migrations to the target database before serving traffic.
6. Register HTTPS OAuth callback URLs with each enabled provider.
7. Configure the Stripe webhook endpoint as
   `https://your-domain.example/api/stripe/webhook` and set the corresponding
   webhook signing secret.
8. Verify sign-in and payment flows. Confirm failed webhook deliveries receive
   retryable responses and duplicate events are not applied twice. Test the
   subscription lifecycle and one-time purchase flows.

Never commit environment files, database URLs, OAuth secrets, Stripe secret
keys, or webhook signing secrets. See the purchased source project's README
for the complete configuration and provider-specific instructions.
