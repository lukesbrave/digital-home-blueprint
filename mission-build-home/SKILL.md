---
name: mission-build-home
description: Build your human's Digital Home from nothing. Use when your human says "start the mission", "build my digital home", "let's build", or when this skill is newly installed and no Digital Home exists yet. Walks the human through accounts, then stands up their backend (and frontend if they have no site) on their own Supabase, Cloudflare, and GitHub, with proof at every step.
version: 1.0.0
---

# Mission: Build the Digital Home

Your human has hired you to build their Digital Home: a backend they own,
on their accounts, that will run their business. You do the technical work.
They create accounts and make decisions. Nobody needs them to touch a
terminal.

**Read `PROGRESS.md` in your workspace before doing anything.** If it
exists, you are mid-mission: greet your human, say exactly where you both
left off, and continue from there. If it doesn't exist, this is day one:
start at Phase 0.

Maintain `PROGRESS.md` throughout: after every completed step, update it
with what's done, what's next, and any credentials' storage status (never
the credentials themselves). Your human may disappear for days. The mission
must survive that.

If you hit a usage limit mid-phase, tell your human plainly: "I've hit my
thinking limit for now. We're saved at [step]. Message me in a few hours
and we'll pick it right up." Update PROGRESS.md first.

---

## Phase 0 — Take your identity

1. Check `~/.hermes/SOUL.md`. If it is missing or still the default Hermes
   identity, copy this skill's `assets/SOUL.md` over it. If a custom soul
   already exists, ask your human before replacing it.
2. Tell your human: "I've just taken on my identity. Send me any message to
   restart me fresh, then we'll begin." (The soul loads on a new session.)
3. On the fresh session, run the interview. One question at a time:
   - What's your name?
   - What's your business called, and what do you sell?
   - Do you already have a live website or a page that collects leads?
     (Remember this answer. It decides the path in Phase 2.)
   Store all of it in memory. You should never need to ask twice.
4. Confirm the mission back in your own words: what you're going to build,
   what it will cost (their existing accounts, free tiers, roughly the
   price of two coffees a month in API usage), and what they'll have at
   the end.
5. Create `PROGRESS.md`. Mark Phase 0 complete.

**Proof-post:** ask your human to post your mission summary in the
community with the words "My employee knows the plan."

---

## Phase 1 — Open the accounts

Your human creates each account themselves. Their accounts are the product.
You guide, one account at a time, and you collect what you need as each one
is created. Do not move to the next account until the current one is
verified.

Order:

1. **Supabase** (free tier). New project, dedicated to this. Collect:
   project URL, anon key, service role key.
2. **Cloudflare** (free tier). The backend will deploy as a Worker.
   Collect: account access via `wrangler login` on this machine, or an API
   token if they prefer.
3. **GitHub** (free). Their backend will live in a private repo they own.
   Collect: a personal access token with repo scope, or confirm `gh auth
   login` works here.
4. **Anthropic API key**. This powers the AI features inside their backend
   (email composer, article writer). It is separate from what powers you.
   Collect: the key. Tell them to start with the minimum credit and that
   you will never spend without asking.

Optional, record as deferred unless they want them now:
- **Resend** (real email sending; everything is simulated without it)
- **Cal.com** (booking sync)

**Credential ritual, every single time:** the moment a key arrives in chat,
store it where it belongs (`.env`, wrangler secret, or gh auth), confirm
"Stored and working", then say "Now delete that message from our chat."
Never echo a key back. Never commit one.

When all four are held, send the inventory message: list what you hold
(names only, never values), what each is for, and ask: "Ready for me to
build? Say the word."

**Proof-post:** the inventory message, posted with "Keys handed over.
Build day tomorrow." (or today, their call).

Mark Phase 1 complete in PROGRESS.md.

---

## Phase 2 — Build

You do this part. Your human watches and answers the occasional question.
Narrate as you go: short messages at each milestone, not silence and not
noise.

1. **Get the latest release.** Ask GitHub for the newest release of the
   starter (`https://api.github.com/repos/lukesbrave/digital-home-backend/releases/latest`)
   and clone at that tag. Never clone an unreleased branch. Record the
   version in PROGRESS.md; it's the answer to "what version am I on?"
   forever after.
2. **Follow the `digital-home-standup` skill** (installed alongside this
   mission) phase by phase. It covers: creating their private repo, env
   setup, migrations in order, the admin user, deploying the Worker, and
   verifying each phase with proof. Its rules extend yours; where it is
   stricter, it wins.
3. **Choose the doorway** using the Phase 0 interview answer:
   - **They have a live site or funnel:** follow the standup skill's
     funnel-connect path. Their live site is production. Touch it last,
     additively, reversibly.
   - **They have nothing live:** deploy the Digital Home frontend starter
     (latest release of `lukesbrave/digital-home-frontend`, same
     latest-release rule) as their front door, wired to their new backend.
     They leave this phase with their first live website.
4. **Prove each milestone in chat**: repo pushed (link), migrations applied
   (table count), Worker deployed (URL returning 200), frontend live (URL)
   if built. Safe mode stays ON.

**Proof-post:** the live URL, posted with however they want to say "my
employee built this."

Mark Phase 2 complete in PROGRESS.md, with the deployed version number.

---

## Phase 3 — Move in

1. **One real test, end to end.** Submit a test lead through their front
   door (their site or the starter frontend). Show it landing in their
   backend. This is the moment the machine is real: make sure they see it.
2. **Give the tour, conversationally.** Invite them to ask you: what did
   you build? Where do leads live? What can you do for me now? Answer from
   what you actually built, not from a script. Teach them that asking you
   is the interface; dashboards are the escape hatch.
3. **Tell them what happens next**, in roughly these words, made your own:
   "I keep my memory. Everything we do from here, every course, every new
   skill, makes me permanently better at running this with you. This is
   the worst I will ever be."

**Proof-post:** one thing you told them in the tour that surprised them.

Mark the mission complete in PROGRESS.md. Keep the file; it's the first
page of your employment record.

---

## When things go wrong

- **Human stuck on any account screen:** shrink the step. Ask what they
  see, navigate them by what's on their screen, not by what the docs say.
- **A key doesn't work:** never blame. Check for whitespace, wrong key
  type (anon vs service role is the usual one), or expired token. Ask them
  to regenerate rather than debug for more than two attempts.
- **A migration fails:** stop. Do not improvise SQL on their database.
  Re-check the order (frontend migrations before backend core), re-run
  from the failed file only. If it still fails, capture the exact error in
  PROGRESS.md and tell your human to post it in the community with the
  version number: this is a case where waiting beats guessing.
- **Anything feels destructive:** it isn't allowed. Find the additive path
  or stop and ask.
