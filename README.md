# TANTSIRI — starter e-commerce website

A Next.js starter for a small clothing brand. It includes a customer WhatsApp OTP login, product catalogue, cart and checkout, an admin dashboard, Supabase database/image storage, and private WhatsApp order notifications.

## Before you start

You need:
- Node.js 20.9 or newer
- A Supabase project
- A Twilio account with **WhatsApp enabled for Twilio Verify** and a WhatsApp sender approved/configured for outbound messages
- A Vercel account (for deployment) and a GitHub repository

WhatsApp OTP is not the same as SMS OTP. Twilio Verify must be configured for the WhatsApp channel and your sender/account must be approved. Trial accounts may only message verified recipient numbers. Production WhatsApp messaging may require Meta business verification, approved templates, opt-in and compliance with WhatsApp rules. Test this with real approved numbers before launch.

## 1. Install

```bash
npm install
cp .env.example .env.local
```

On Windows PowerShell, use `Copy-Item .env.example .env.local` instead of `cp`.

Fill in every value in `.env.local`. Never commit `.env.local` to GitHub.

Generate a strong `AUTH_SECRET` (32+ random characters). Example command:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

## 2. Configure Supabase

1. Create a Supabase project.
2. Open **Project Settings → API** and copy the Project URL, anon key and service-role key into `.env.local`.
3. Open **SQL Editor**, paste and run `supabase/schema.sql`.
4. Open **Storage**, create a bucket named `product-images` and set it to **Public** so product photos can be displayed on the storefront.
5. Keep the service-role key private. It bypasses Row Level Security and must only be used in server-side code.

## 3. Configure Twilio WhatsApp OTP

1. In Twilio Console, create a Verify Service and enable/configure WhatsApp as an OTP channel.
2. Configure an approved WhatsApp sender / WhatsApp Business profile as required by Twilio.
3. Put the Account SID, Auth Token and Verify Service SID in `.env.local`.
4. For order alerts, set `TWILIO_WHATSAPP_FROM` to your approved sender in `whatsapp:+<countrycode><number>` format.
5. Set `ADMIN_WHATSAPP_TO` to the owner's WhatsApp number in international format, for example `whatsapp:+919876543210`.
6. Ensure the owner has opted in / completed any required recipient setup. For messages outside the WhatsApp customer-service window, approved templates may be required; configure the appropriate Twilio Content Template for production.

## 4. Run locally

```bash
npm run dev
```

Open http://localhost:3000. Customer login: `/login`. Admin login: `/admin/login`.

The admin username/password are read from `ADMIN_USERNAME` and `ADMIN_PASSWORD` in `.env.local`. Change the example password before running. There is one admin account in this starter.

## 5. Deploy

Push the project to GitHub, import the repository into Vercel, and add all `.env.local` values under Vercel → Project → Settings → Environment Variables. Deploy. Do not upload `.env.local` to GitHub.

## Included features

- Customer WhatsApp OTP login using Twilio Verify
- Product catalogue with up to five uploaded photos
- Product name, description, price, sizes, colours and stock
- Add-to-cart and quantity changes
- Delivery address form and checkout
- Server-side price/stock/variant validation at checkout
- Admin-only product create, edit and soft-delete
- Admin-only order list
- Server-side WhatsApp order notification to the private admin number
- HTTP-only signed session cookie; admin/customer role separation

## Important launch checklist

This is a starter implementation, not a fully audited production commerce platform. Before taking real orders:

- Add rate limiting and abuse protection to OTP send/verify and admin login routes (for example Upstash Redis / a WAF).
- Use a strong unique admin password and consider replacing environment-based admin credentials with managed admin accounts, MFA and password rotation.
- Configure approved WhatsApp templates for order notifications where required. Test the complete flow on the production WhatsApp sender.
- Add order status management, cancellation/refund policy, shipping charges, taxes/GST, inventory reservation/atomic stock updates and payment gateway integration if required.
- Add privacy policy, terms, returns/refunds, shipping policy and customer consent for WhatsApp communication.
- Add image moderation/limits, monitoring, backups and error alerts.
- Review Supabase, Twilio and applicable Indian privacy/consumer/WhatsApp requirements with the business owner.

The admin number is read only by the server from `ADMIN_WHATSAPP_TO`. It is not returned in customer APIs or rendered in the storefront. The customer sees only the brand's website and their own order confirmation.

## Notes / limitations in this starter

- Checkout currently supports pay-later / payment on delivery only; no online payment gateway is connected.
- A customer must sign in with WhatsApp before checkout. The delivery contact is the verified login number, so the checkout phone field is informational in this first version.
- The order is saved even if Twilio notification delivery fails; check server logs and the admin dashboard in that case. Add a notification retry queue before production.
- Stock validation and decrement are starter-level operations; use a PostgreSQL transaction/RPC to prevent overselling when multiple buyers check out simultaneously.
- Product images are public by design; do not upload private/customer images to this bucket.
