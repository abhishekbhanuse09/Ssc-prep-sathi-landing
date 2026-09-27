# SSC Prep Sathi — Landing Page

Pre-launch landing page for SSC Prep Sathi with a real pre-registration form:
Next.js (App Router) + TypeScript + Tailwind + Supabase + Resend.

## What this does

1. Student fills the form (name, problem, class, email).
2. Client + server validate it (Zod schema shared both places).
3. Server inserts the row into Supabase using the **service-role key**
   (server-only — never sent to the browser).
4. After the DB insert succeeds, the server:
   - emails **you** (admin) the new registration
   - emails the **student** a welcome message
5. If an email fails, the registration is still saved — the student never
   sees an email-provider error. Failures are logged server-side and the
   `admin_notified` / `welcome_sent` columns track what actually went out.

## 1. Install

```bash
npm install
```

## 2. Set up Supabase

1. Create a project at [supabase.com](https://supabase.com) (free tier is fine).
2. Open the SQL editor and run `supabase/schema.sql` from this repo.
   - This creates the `registrations` table.
   - Row Level Security is enabled with **no public policies** — the anon
     key cannot read/write this table at all. Only the service-role key
     (used server-side only) can.
3. Go to Project Settings → API and copy:
   - `Project URL` → `SUPABASE_URL`
   - `service_role` key → `SUPABASE_SERVICE_ROLE_KEY` (keep this secret)

## 3. Set up Resend (email)

1. Create a free account at [resend.com](https://resend.com).
2. Get an API key from **API Keys** in the dashboard → `RESEND_API_KEY`.
3. For real sending, verify a domain under **Domains** and use an address
   on it for `EMAIL_FROM` (e.g. `SSC Prep Sathi <hello@sscprepsathi.com>`).
   - For quick testing before you own a domain, you can use
     `onboarding@resend.dev` as `EMAIL_FROM`, but Resend restricts who that
     can send to — a verified domain is required before real students can
     receive mail reliably.
4. Set `ADMIN_NOTIFICATION_EMAIL` to whatever inbox you want new
   registrations sent to.

## 4. Environment variables

```bash
cp .env.example .env.local
```

Fill in the four values from steps 2–3.

## 5. Run locally

```bash
npm run dev
```

Visit http://localhost:3000

## 6. Before deploying

```bash
npm run typecheck
npm run lint
npm run build
```

All three should pass cleanly.

## 7. Deploy (Vercel)

1. Push this repo to GitHub.
2. Import it in Vercel.
3. Add the four environment variables from `.env.example` in Vercel's
   Project Settings → Environment Variables.
4. Deploy.

## Notes / things to know

- **The welcome email only sends if `contact` is an email address.** The
  form currently only collects email (no phone/WhatsApp), so this always
  applies — it's future-proofed in `src/lib/email.ts` in case you add phone
  input later.
- **Rate limiting is in-memory** (`app/api/register/route.ts`), which stops
  accidental double-submits on a single server instance but won't hold up
  against a determined abuser or a multi-instance deployment. If this
  matters to you, swap in a real rate limiter (e.g. Upstash Redis) later —
  the code is isolated in one function (`getClientKey`) to make that easy.
- **No admin dashboard is included.** View submissions directly in the
  Supabase Table Editor for now, as the brief asked to avoid building
  unnecessary dashboards/auth for v1.
- Nothing on this page claims features exist yet, or fabricates users,
  testimonials, or stats — the concept preview and future-features section
  are explicitly labeled as not-yet-built.
