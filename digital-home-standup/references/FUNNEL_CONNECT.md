# Connecting a Live Funnel to the Capture Endpoint

The CRM's front door is one endpoint. Everything the funnel needs to do is
POST to it when a lead completes the quiz (or any form step worth capturing).

## The endpoint

```
POST {backend-url}/api/crm/capture
Header: x-capture-key: <crm_capture_key>
Content-Type: application/json
```

`crm_capture_key` was generated during the upgrade install (Phase 2, step 4
of the upgrade skill) and lives in the `backend_settings` table. Read it from
there; never regenerate it casually — regenerating breaks every existing
integration using it.

### Body

```json
{
  "email": "lead@example.com",        // required, validated
  "name": "Sam Smith",                 // or first_name / last_name
  "phone": "+1 555 0100",              // optional
  "source": "quiz-funnel",             // where this lead came from
  "page": "/quiz/results",             // capture page
  "form": "quiz",                      // form identifier
  "tags": ["quiz", "quiz-outcome-a"],  // drives workflow auto-enrollment
  "custom": {                          // quiz answers go here
    "q1_biggest_challenge": "...",
    "q2_budget": "...",
    "quiz_outcome": "A"
  },
  "workflow_id": "..."                 // optional: enroll directly into one workflow
}
```

### Response

`{ "ok": true, "lead_id": "...", "created": true }` — `created: false` means
the lead already existed and was updated (capture upserts by email, so
repeat submissions never create duplicates).

### Behavior you get for free

- Upsert by email, activity logged on the lead timeline
- A deal auto-filed into the sales pipeline ("New", or "Nurturing" if a tag
  trigger enrolled them into a sequence)
- Tag-based workflow enrollment: any active workflow with a matching tag
  trigger picks the lead up automatically

## Rules for a live funnel

1. **The capture key never appears in browser JavaScript.** CORS on the
   endpoint is open, but the key is a server secret. Call capture from:
   - the funnel's existing server-side submit handler, or
   - a small serverless function the funnel already posts to, or
   - the form tool's native webhook feature (most support a custom header),
     or Zapier/Make if that's what the funnel already uses.
   If the funnel is pure static client-side JavaScript with no server at
   all, add a tiny proxy function (Cloudflare Worker, Vercel function)
   rather than exposing the key. This is non-negotiable.

2. **Dual-write, don't reroute.** The funnel keeps doing exactly what it
   does today (its dashboard, its notifications, its sheet, whatever).
   Capture is an *additional* destination. Removing the old path is a
   separate decision for another day, made by the member, not by you.

3. **Fail open.** Wrap the capture call so a CRM outage can never block or
   slow the funnel's own submit flow. Fire-and-forget with a short timeout,
   or send from an async webhook.

4. **Never name a funnel field `website`.** The capture endpoint treats a
   filled `website` field as a bot honeypot and silently drops the
   submission (it still returns `ok: true`, which makes this brutal to
   debug). If the quiz genuinely collects a URL, send it inside `custom`.
   Conversely: if the funnel form wants bot protection, a hidden `website`
   field is the supported trick.

5. **Test before wiring.** Prove the endpoint from the command line first:

   ```bash
   curl -s -X POST "$BACKEND_URL/api/crm/capture" \
     -H "x-capture-key: $CAPTURE_KEY" \
     -H "Content-Type: application/json" \
     -d '{"email":"test+standup@example.com","name":"Test Lead","source":"connect-test","tags":["quiz"],"custom":{"note":"delete me"}}'
   ```

   Confirm the lead appears in `/crm/leads`, then delete it, then wire the
   funnel and repeat through the real quiz.

## Mapping quiz data

- One tag per meaningful segment: `quiz` on everyone, plus an outcome tag
  (`quiz-outcome-a`) if the quiz has result branches. Workflows trigger on
  tags, so tags are the routing layer — design them for the sequences the
  member wants to send, not for reporting (custom fields cover reporting).
- Full answers belong in `custom` with stable snake_case keys. They show on
  the lead timeline and give the AI composer real material to personalize
  sequences with.
- `source` stays constant per funnel (`quiz-funnel`); `page`/`form`
  distinguish steps if the funnel captures at more than one point.
