# LLM-OVERVIEW — notary
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Mobile notary public platform providing real-time distance-based pricing quotes, online appointment scheduling, Stripe payment processing (credit cards and Afterpay), automated SMS and email notifications, California legal compliance tracking (journal logging and $15 legal signature fee caps), document verification, and notary availability management. Built on Next.js 14 App Router, React 18, TypeScript, Tailwind CSS, Prisma ORM, and PostgreSQL. Deployed to Vercel Serverless Functions with a Homelab PostgreSQL database backend (migrated from Railway).

## Machine & Host Ownership
Status probe disabled (no live status probe output available). According to `AGENTS.md`, the platform operates across two runtime environments:
- **Vercel Serverless Functions**: Hosts the Next.js 14 web application, server-side rendering, and serverless API endpoints (`app/api/*`).
- **Homelab Database Host**: Local infrastructure host running PostgreSQL 15 with PgBouncer connection pooling in Docker (`homelab/docker-compose.yml`), managing database storage and automated 30-day backups.

## What is actually built
- **Pricing & Distance Calculation Subsystem (`lib/pricing.ts`, `lib/distance.ts`)**:
  - Calculates dynamic travel and notary fees based on client location, signature counts, time of day, and urgency.
  - Base location: Thai Town, Los Angeles (`90027`, `34.0978, -118.2930`) covering up to 50 miles in LA County.
  - Base fees: $15 per signature (California statutory maximum limit) + $75 mobile travel fee (first 10 miles included; $0.50 per additional mile).
  - Time & urgency surcharges: After-hours (6PM–9AM: +$50), Weekend (+ $25), Holiday (+ $100), Same-day (+ $50), 2-Hour Rush (+ $100), Emergency/ASAP (+ $150).
  - IRS Mileage Deduction Tracker: Calculates `distanceInMiles * IRS_RATE` per appointment for operator tax tracking.
  - Distance engine: Uses Google Maps Distance Matrix API with a Haversine formula fallback.

- **Booking & Scheduling Flow (`app/book/`, `components/DateTimePicker.tsx`)**:
  - 24/7 calendar picker filtering slots based on `NotaryAvailability` weekly rules and `BlockedDate` blackouts.
  - Form steps capture service type (Acknowledgment, Jurat, Loan Signing, etc.), location, document count, and special instructions.

- **Payment & Checkout Engine (`app/api/checkout/`, `app/api/create-payment-intent/`, `app/api/webhooks/stripe/`, `lib/stripe.ts`)**:
  - Integrated Stripe payments supporting standard credit cards and Afterpay 4-installment plans.
  - Lazy/runtime initialization of Stripe SDK in `lib/stripe.ts` to prevent build-time static render failures.
  - `/api/webhooks/stripe` handles `payment_intent.succeeded` events, updates `PaymentStatus` to `COMPLETED`, generates unique secure `cancellationToken` strings, and dispatches confirmations.

- **Notifications & Cron Reminders (`lib/email.ts`, `lib/sms.ts`, `app/api/cron/send-reminders/`)**:
  - Email dispatch via Resend API (`resend` 3.2.0) for transactional booking confirmations, receipts, and cancellations.
  - SMS dispatch via Twilio API (`twilio` 5.10.6).
  - Vercel Cron route (`/api/cron/send-reminders/`) triggers 24-hour and 1-hour automated SMS and email reminders prior to scheduled appointments.

- **California Compliance & Legal Journal (`prisma/schema.prisma`, `app/dashboard/`)**:
  - Database model `JournalEntry` enforces California Government Code requirements for notarizations: notarization date/time, act type, document date/type, signer name & address, identification method/number/issuer (e.g. CA Driver License, Passport), witness details, and fee logged.
  - Secure client document metadata storage (`Document` model) with verification flags.

- **Notary Operator Dashboard & Self-Service Portal (`app/dashboard/`, `app/receipts/[id]/`, `app/appointments/cancel/`)**:
  - Admin dashboard for appointment management, status updates (`PENDING`, `CONFIRMED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`, `NO_SHOW`), revenue metrics, client documents, and journal compliance entry creation.
  - Printable client receipt system (`app/receipts/[id]`).
  - Token-authenticated client self-service cancellation and rescheduling interface (`/appointments/cancel`).

- **Database & Data Layer (`prisma/schema.prisma`)**:
  - Prisma ORM 5.22.0 models: `User`, `Account`, `Session`, `VerificationToken`, `Appointment`, `Document`, `JournalEntry`, `NotaryAvailability`, `BlockedDate`, `PricingRule`.

- **Homelab & Deployment Infrastructure (`homelab/`, `scripts/`)**:
  - `homelab/docker-compose.yml`: PostgreSQL 15 + PgBouncer setup.
  - Automated database scripts: `homelab/scripts/setup.sh`, `init-db.sh`, `backup.sh`, `restore.sh`.
  - Migration utilities: `scripts/migrate-from-railway.sh`, `validate-migration.sh`, `setup-vercel.sh`.
  - Legacy deployment files (`railway.toml`, `Dockerfile`, `railway/`) remain in repo history from prior Railway setup before Vercel + Homelab migration.

## Canonical entry points
- **Web Pages (Next.js 14 App Router)**:
  - `app/page.tsx`: Landing page with instant dynamic quote calculator and service overview.
  - `app/pricing/page.tsx`: Detailed pricing calculator and fee structure explanation.
  - `app/services/page.tsx`: Overview of notary acts and California legal compliance information.
  - `app/book/confirm/page.tsx`: Pre-checkout booking review page.
  - `app/book/success/page.tsx`: Post-payment appointment confirmation page.
  - `app/appointments/cancel/page.tsx`: Token-authenticated booking cancellation/reschedule interface.
  - `app/dashboard/page.tsx`: Operator admin dashboard and legal journal management.
  - `app/receipts/[id]/page.tsx`: Customer receipt view page.

- **API Routes (`app/api/`)**:
  - `app/api/pricing/route.ts`: Pricing calculation endpoint (distance, surcharges, signature fee).
  - `app/api/checkout/route.ts`: Stripe Checkout session creation endpoint.
  - `app/api/create-payment-intent/route.ts`: Direct PaymentIntent creation endpoint.
  - `app/api/webhooks/stripe/route.ts`: Stripe webhook event handler.
  - `app/api/cron/send-reminders/route.ts`: Vercel cron route for 24h/1h SMS/email reminder delivery.
  - `app/api/health/route.ts`: Health check endpoint (DB connectivity, environment state).
  - `app/api/appointments/cancel/route.ts`: Appointment cancellation API endpoint.
  - `app/api/appointments/verify/route.ts`: Token verification endpoint.
  - `app/api/receipts/[id]/route.ts`: Receipt lookup API.

- **Core Business Logic & Utilities (`lib/`)**:
  - `lib/pricing.ts`: Pricing logic engine, fee constants, and surcharge calculators.
  - `lib/distance.ts`: Google Maps Distance Matrix API caller with Haversine fallback.
  - `lib/stripe.ts`: Runtime initialized Stripe client.
  - `lib/prisma.ts`: Prisma Client singleton handler.
  - `lib/email.ts`: Transactional email dispatcher (Resend).
  - `lib/sms.ts`: SMS message dispatcher (Twilio).
  - `lib/validations.ts` & `lib/validation.ts`: Zod schema validation rules.

- **Schema & Infrastructure**:
  - `prisma/schema.prisma`: Database schema and enum definitions.
  - `homelab/docker-compose.yml`: Local PostgreSQL 15 & PgBouncer container specification.
