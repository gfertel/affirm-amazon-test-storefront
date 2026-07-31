# affirm-amazon-test-storefront

Internal test tool for Affirm's Amazon UK GIL (Global Installments Lending)
Pay-By-Affirm integration. Talks to Affirm's **production** API using a test
merchant account — not a general-purpose demo, and not affiliated with or
endorsed by Amazon.

This repo holds only the static frontend (`index.html`). The Cloudflare
Worker that fronts it (gating, auth, API proxying) is deployed and
maintained separately, and is intentionally not mirrored here — no
credentials, routes, or gating mechanics live in this public repo.

**Always use the Worker URL, never this repo's raw GitHub Pages URL.** The
Worker is the password-gated front door; `index.html` only works when loaded
through it (the session cookie and API calls both require same-origin).
Loaded directly from GitHub Pages, the page just shows a "wrong URL" notice
and has no working "Pay by Affirm" button — the API rejects cross-site
requests from that origin.

## Setup

The fronting Cloudflare Worker needs these set as Worker secrets — never in
this repo:

| Secret | What it is |
|---|---|
| `SITE_PASSWORD` | Shared password handed to testers |
| `SESSION_SECRET` | Random signing key, e.g. `openssl rand -hex 32` |
| `GIL_API_PUBLIC_KEY` / `GIL_API_PRIVATE_KEY` | Test merchant's Basic Auth key pair |
| `GIL_CLIENT_ID` | `client_id` value for that key pair |

## Using it

1. Go to `https://<worker>.workers.dev/amazon-gil/`, enter the shared password.
2. Edit the cart to a specific total, adjust offer settings if needed
   (installment terms, interest-bearing vs 0%, Affirm-funded vs
   merchant-funded), then proceed to checkout.
3. Fill in a UK delivery address (nothing pre-filled) and continue — this
   calls the real production `create_loan_application` endpoint.
4. Complete Affirm's real hosted checkout flow as the tester would.
5. You're redirected back automatically; HOLD fires on its own to authorize,
   then a **Void transaction** button is available to release it.
6. `#/manual` lets you fire HOLD/RELEASE directly by id if you lose the
   session mid-flow. Every result screen has a debug panel with the raw
   request/response JSON.

## Known limitation

This repo, and the GitHub Pages site built from it, are public — GitHub
Pages has no access control, on any plan short of GitHub Enterprise. The
Worker makes the raw page inert rather than pretending otherwise: its API
routes only ever accept same-origin requests from the Worker's own domain,
so a copy loaded straight from GitHub Pages has nothing it can actually do.
