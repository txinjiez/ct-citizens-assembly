# ct-citizens-assembly

Marketing and member-portal site for the Connecticut Citizens' Assembly on Property Taxes. Static HTML/CSS/JS deployed to Netlify, with a Supabase-backed members area for the dashboard and login pages.

## Overview

The site is a no-build static site: ~16 standalone HTML files, one shared JS file (`js/supabase-config.js`), and assets under `media/`. There is no framework, no bundler, and no templating — each page is self-contained with its own inline `<style>` block and a copy-pasted nav/footer.

Two flows coexist:

- **Public brochure** (the working flow): hero, about, team, press, donors, resources, committee pages, and a landing page with an embedded invitation PDF and a Google Form contact iframe. `register.html` funnels participant applications to a Yale Qualtrics survey.
- **Member area** (wired, currently inert in production): `login.html` and `dashboard.html` talk to Supabase for auth, session registration, and a personal message archive. The schema is documented as commented-out SQL in `js/supabase-config.js`.

Dependencies are all loaded via CDN: Google Fonts (Playfair Display, Public Sans), Google Tag (`G-CYXQ157GLP`), and `@supabase/supabase-js@2`.

### Layout

```
index.html               Landing page
about.html               What is a citizens' assembly
team.html                Team + advisors (bios in hardcoded JS object)
press.html               News
donors.html              Donor acknowledgements
resources.html           General resources + IEGC meeting minutes
research-committee.html  Committee stubs (under construction)
scale-committee.html
stakeholders-committee.html
register.html            Participant signup — links out to Qualtrics
survey-1.html            UI mockup of a survey (non-functional)
login.html               Supabase auth
dashboard.html           Member area (auth-gated, Supabase-backed)
js/supabase-config.js    Supabase client + schema documentation
media/                   Logos, headshots, PDFs, meeting minutes
netlify.toml             Headers + pretty-URL rewrites
prototype/               Earlier design iteration (kept for reference)
```

## Getting Started

No install step. Clone the repo and serve the directory with any static server:

```bash
# Option 1: Python (simplest)
python3 -m http.server 8000

# Option 2: Netlify CLI (honors /dashboard and /login pretty URLs)
npx netlify-cli dev
```

Then open http://localhost:8000.

To make the members area work locally, edit `js/supabase-config.js` and replace the placeholder values:

```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

with the project's real Supabase URL and anon key. The SQL schema (tables, RLS policies, seed data) needed in the Supabase project lives as commented SQL at the bottom of the same file.

## Deploy

Hosted on Netlify; pushes to `main` deploy automatically. `netlify.toml` configures:

- Security headers (`X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`)
- Pretty URL rewrites: `/dashboard` → `/dashboard.html`, `/login` → `/login.html`

There is no build command — Netlify serves the repo root as-is.

## Testing

There is no test suite. CI runs `.github/workflows/validate.yml` on push and PR to `main`:

- `html-validate *.html` — warnings only (`continue-on-error: true`)
- A grep that checks `src=`/`href=` references against files under `media/`

Verify changes manually by serving locally (see Getting Started) and walking the affected pages in a browser.

## TODOs

- **Wire up Supabase credentials.** `js/supabase-config.js` still has `'YOUR_SUPABASE_URL'` / `'YOUR_SUPABASE_ANON_KEY'` placeholders. Until these are real, login, signup, dashboard session-registration, and the contact-form persistence all silently no-op.
- **Apply the schema.** Run the commented-out SQL in `js/supabase-config.js` (or a migration derived from it) against the Supabase project to create `profiles`, `assembly_sessions`, `session_registrations`, and `form_submissions` with their RLS policies and the `handle_new_user()` trigger.
- **Reconcile schedule data.** Marketing pages list assembly dates Jul 11–Sep 26; the seed SQL in `js/supabase-config.js` has a different Jun 6–Aug 22 list. Pick one and update the other.
- **Fix the broken contact handler in `index.html`.** The `.btn-submit` click handler around line 1862 targets a form that no longer exists (replaced by a Google Form iframe), and throws `TypeError` on every landing-page load.
- **Unhide the register tab in `login.html`.** The register tab button is commented out (around line 297), so the signup UI is unreachable through the page even though `register()` and the form markup are implemented.
- **Rename or repurpose `register.html`.** It is not the account-signup page — it is a Qualtrics funnel. The name collides with the hidden register tab in `login.html`.
- **Link real PDFs in the dashboard.** `dashboard.html` "Participant Resources" cards have `href="#"`; the actual 2-pager PDF exists at `media/CT-Citizens-Assembly-2-Pager.pdf`.
- **Remove dead pages.** `governance.html` (old variant of team.html, only referenced by a commented-out nav line) and `content.html` (internal copy reference doc) are orphans.
- **Decide what to do with `prototype/`.** It references `js/` and `media/` paths that don't exist inside that directory, so it 404s if visited.
- **Add a `.gitignore`.** Currently missing; `~$CA-website-text.docx` (a Word lock file) is checked in as a result.
- **Consider a shared nav include.** The 17-line nav block is copy-pasted across 14 pages and has already drifted (some pages comment out items others still show). A simple JS include or moving to a static-site generator would eliminate the drift.
- **Drop the Supabase SDK from pure marketing pages.** Pages like `donors.html` and `about.html` load ~80–100 KB of Supabase SDK only to maybe rewrite "Login" → "My Account" in the nav. A `localStorage` flag set at login would be enough.
- **Fix the CI missing-media check.** The bash one-liner in `.github/workflows/validate.yml` sets `missing=1` inside a piped subshell, so it never propagates and the step always exits 0.
