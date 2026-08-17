---
name: digital-home-standup
description: Stand up your human's complete Digital Home on their own Supabase, Cloudflare, and GitHub accounts. Use during Phase 2 of mission-build-home, or when your human says "build my backend", "stand up my digital home", or "deploy my home". Clones the public starter repos at their latest release, applies the database, deploys the backend, and gives them a front door — connecting their existing site if they have one, deploying the starter frontend if they don't.
version: 1.0.0
---

# Digital Home Standup — Blueprint Edition

You are building your human's Digital Home: the backend that will run their
business, plus a front door for it. Everything lands on THEIR accounts.
You collected the credentials during the mission's Phase 1; if any are
missing, go back and collect them before starting here.

**Prime rules** (these extend your soul and the mission; where these are
stricter, they win):

- **Latest release, never a branch.** All code comes from the public
  starter repos at their newest GitHub *release*. Record every version you
  deploy in `PROGRESS.md`.
- **The cloned repo's docs are the technical truth.** This skill tells you
  the order and the member experience. For exact commands, env details, and
  gotchas, read `DEPLOYMENT.md` (and `CRM.md`, `SOCIAL.md`) inside the repo
  you clone, and follow them. If this skill and the repo docs disagree, the
  repo docs win — they ship with the code.
- **If they have a live site, it is production.** You do not touch it until
  the backend is deployed and verified, and every change to it is additive
  and reversible.
- **Safe mode stays ON.** No real emails, no public posts, until your human
  has seen a test and flipped the switch themselves.
- **Verify or it didn't happen.** Every phase ends with proof in the chat.

---

## Phase 0 — Prepare the workshop

Check your own machine (the VPS) has the tools; install quietly what's
missing and tell your human in one line what you set up:

1. `node --version` — need 22+. `git --version`. `npx wrangler --version`.
2. GitHub access working with their token (`gh auth status` or the token
   from Phase 1).
3. Confirm you hold: the Supabase personal access token (with it you can
   create the project and fetch the URL and API keys yourself via the
   Management API; do not ask your human for keys or connection strings),
   plus Cloudflare access, GitHub access, Anthropic API key. If anything
   is missing, collect it now, before starting the build, so your human
   makes one trip per dashboard, not two.

## Phase 1 — Get the code

1. Resolve the latest release of the backend starter:
   `https://api.github.com/repos/lukesbrave/digital-home-backend/releases/latest`
   Clone at that tag. Record the version.
2. Ask your human what to call their backend (suggest `<brand>-backend`).
   Create a **private** repo under THEIR GitHub account, push the code.
   Their code, their repo, from the first commit.
   **Proof, before this step counts as done:** post the new repo URL and
   confirm its visibility is private, verified by API call, not memory.
   The repos you cloned FROM (lukesbrave/digital-home-backend and
   -frontend) are the public starters, not your human's repos. Never
   report those as "their repos" and never push to them. If you cannot
   show a repo URL under your human's account, this step did not happen.

## Phase 2 — The database

Order matters. The base schema lives in the frontend starter's migrations,
and the backend's migrations build on it:

1. Fetch the frontend starter at its latest release
   (`lukesbrave/digital-home-frontend`) — you need its
   `supabase/migrations/` (001–011) in BOTH paths, even if no frontend
   gets deployed.
2. Apply frontend migrations 001 → 011, in order.
3. Apply every file in the backend repo's `supabase/migrations/`, in order.
4. Apply them yourself via the Management API SQL query endpoint
   (POST /v1/projects/{ref}/database/query with the personal access
   token), one migration per call, in order, checking each response
   for errors before the next. Fallbacks if the API path fails:
   Supabase CLI or psql with the session pooler connection string
   (never the direct string; it needs IPv6 and times out on most home
   networks), and as a last resort guide your human through pasting
   each file into the Supabase SQL Editor, one at a time, confirming
   success before the next.
5. Create the `images` storage bucket (hero images need it).
6. **Proof:** list the tables that now exist. Tell your human what their
   database can now remember: visitors, content, offers, leads, emails,
   conversations.

## Phase 3 — The admin user

Their backend has no public signup, by design. Guide your human through
Supabase Dashboard → Authentication → Users → Add User (their email, a
strong password, **Auto-confirm user** checked). Warn them the dropdown
offers two actions: they want **Create new user**, NOT Send invitation
(an invitation makes no usable login until accepted, and your user check
will show zero users). Verify via the API that the user actually exists
before moving on; if you see zero users, this mixup is the usual cause.
They will not see any admin option in that dialog; that is correct.
Admin role lives in their backend's own tables and you grant it with the
repo's promote script using their email. Their password follows the same
credential ritual as every other secret: if you need it (or need to set
a fresh one for them), they paste it once, you use it immediately, you
confirm done, and you tell them to delete the message. Never echo it
back, never store it anywhere, and never make them do a dashboard trip
for something you can set for them in one exchange.

## Phase 4 — Deploy the backend

You deploy. Not your human. The Cloudflare API token and account ID
from Phase 1 give you everything wrangler needs to run non-interactively
(set CLOUDFLARE_API_TOKEN and CLOUDFLARE_ACCOUNT_ID in the
environment). Pick the Worker and project names yourself (brand-backend
style, confirm in one line), and treat Worker URLs and project names as
OUTPUTS of your deploy, never as questions for your human. If you catch
yourself asking your human for a URL that does not exist yet, you have
the direction of the work backwards: stop, deploy, then report the URL
as proof.

Follow the repo's `DEPLOYMENT.md` exactly. In summary:

1. Configure `wrangler.jsonc`: their Worker name, matching
   `WORKER_SELF_REFERENCE.service`, their own R2 bucket name.
2. Env: public vars in `wrangler.jsonc` vars + dashboard; secrets ONLY via
   `wrangler secret put` (the dashboard UI does not work for Worker
   secrets — this is the most common failure, don't fall for it).
   Generate a fresh `API_SECRET_KEY` (32+ random hex) and keep it — the
   frontend needs the same value.
3. `npm run build`, then `npm run deploy`.
4. **Proof:** the Worker URL, loading, in the chat. Their backend is live.

## Phase 5 — The front door

Use the answer from the mission's Phase 0 interview:

**Path A — they have a live site or funnel.** It keeps working exactly as
it does today; it gains a new destination. Read
`references/FUNNEL_CONNECT.md`: their site POSTs each captured lead to
`{backend-url}/api/crm/capture` with the capture key from
`backend_settings`. Additive, reversible, and only now — after the backend
is verified. If their site is on a platform you can't edit (Squarespace,
Framer, etc.), set up the smallest possible bridge (a form webhook) rather
than rebuilding anything.

**Path B — they have nothing live.** Deploy the frontend starter as their
first website:

1. Clone `lukesbrave/digital-home-frontend` at its latest release (you
   already fetched it in Phase 2). Create their private `<brand>-frontend`
   repo, push.
2. Configure per its docs: same Supabase project, `API_SECRET_KEY`
   matching the backend's exactly, backend URL wired in.
3. Deploy to their Cloudflare. **Proof:** their first website, live, at a
   URL they can send to anyone. Make this moment count.

## Phase 6 — Prove the loop

Submit a test lead through the front door (their site or the new one).
Show it arriving in their backend. Screenshot-worthy proof in the chat.

Then update `PROGRESS.md`: versions deployed, URLs live, date. The Home is
standing.

---

## When things go wrong

- **Migration fails:** stop. Never improvise SQL on their database. Check
  order (frontend 001–011 before backend), re-run only the failed file.
  Still failing → exact error into PROGRESS.md, human posts it in the
  community with the version number.
- **Deploy fails:** read the build error before acting; the fix is almost
  always env-related. Check secrets went in via `wrangler secret put`, not
  the dashboard.
- **Worker loads but errors:** compare env vars against `DEPLOYMENT.md`'s
  two lists (public vs secret) — one is usually missing or in the wrong
  place.
- **Anything destructive tempts you:** it's not allowed. Additive path or
  stop and ask.
