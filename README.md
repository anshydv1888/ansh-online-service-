# Ansh Online Service — Full-stack starter

This project advances the visual prototype into an Express + PostgreSQL starter with:
- Server-side password login and session cookies.
- Roles: `retailer`, `distributor`, `distributor_head`, `super_admin`.
- Admin-only user listing, status changes and audit logs.
- PostgreSQL schema for users, wallet balances/ledger, service catalogue, pricing, requests and audit history.
- All 50 services seeded as **disabled** until each is verified and authorized.
- OTP, real wallet top-ups and service submissions explicitly disabled until providers are configured.

## Requirements
- Node.js 20+
- PostgreSQL 14+
- HTTPS and a managed host for production.

## Local setup
1. Install PostgreSQL and create an empty database named `ansh_online_service`.
2. Copy `.env.example` to `.env` and update `DATABASE_URL` and `SESSION_SECRET`.
3. Run the schema against your database:
   ```sh
   psql "$DATABASE_URL" -f schema.sql
   ```
4. Install dependencies:
   ```sh
   npm install
   ```
5. Create the first owner account:
   ```sh
   node scripts/create-admin.js
   ```
   Use a strong, unique password of at least 12 characters. Run this script only once for the first admin.
6. Start the application:
   ```sh
   npm start
   ```
7. Open `http://localhost:3000`.

## Custom domain deployment checklist
1. Choose a Node.js host that supports environment variables, HTTPS, persistent PostgreSQL and logs.
2. Create a production PostgreSQL database; apply `schema.sql` through a secure migration step.
3. Deploy this repository from a private Git repository or upload it to the host.
4. Set `NODE_ENV=production`, `DATABASE_URL`, and a cryptographically random `SESSION_SECRET` (32+ characters). Do not upload `.env` to Git.
5. Set `ALLOW_INSECURE_LOCAL_COOKIES=false` in production (production cookies are secure by default in code); use HTTPS end-to-end.
6. Point your purchased domain's DNS records to your host and enable automatic TLS/HTTPS.
7. Run `node scripts/create-admin.js` once through a secure terminal, then remove any temporary shell access where appropriate.
8. Test login, logout, role boundaries, backups, session expiry, rate limits, monitoring and restore procedures before public launch.

## OTP
OTP endpoints return `503` by design. Choose an SMS/OTP vendor, register sender/template as required, generate OTPs server-side, store only a hash, enforce a short expiry and attempt limits, rate-limit by account/IP, and never log OTP values. Then implement request and verification with provider credentials stored only in server environment variables. Consider password + OTP MFA for the Super Admin.

## Payments and wallet safety
Real top-ups return `503` by design. Integrate an approved payment gateway. Create orders server-side, verify signed webhooks, verify amounts/currency/order IDs, enforce idempotency, and credit wallets only after verified success in a database transaction. Keep a proper double-entry or append-only ledger and reconciliation process. Never collect/store UPI PINs, never trust a browser redirect as payment confirmation, and never let users edit balances directly.

## Government/provider APIs
Do not enable a service until you have the relevant authorization, provider contract/API credentials and tested integration. Keep provider cost, government fee, customer-facing service fee, tax and each role's commission separate. The `enabled` field defaults to false. For services without an authorized API, provide a clearly labelled assistance workflow instead of claiming automatic submission.

## Production hardening still required
This is a starter, not a certified production service. Before public use, add CSRF protection for cookie-authenticated mutations, a strict Content Security Policy, schema migrations, robust validation, email/mobile verification, password reset, account registration/approval flows, MFA, security monitoring, database backup/restore tests, privacy/retention policies, and a review by a qualified developer. Do not use the demo UI role selector as access control; server middleware is the authority.


## UPI QR and account ID pricing update
- The supplied QR image is available at `/assets/upi-qr.jpg`.
- UPI ID shown: `9278474298@fam`.
- Displayed account ID charges: Retailer ₹0, Distributor ₹50, Distributor Head ₹150.
- These are display-only in demo mode. Do not activate accounts or credit wallets based on screenshots or unverified transfers. Add a payment gateway/webhook and server-side verification before enabling real wallet funding.
