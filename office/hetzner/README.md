# The office — Hetzner sovereign edition

Your employee needs a home that never sleeps: a small rented server that is
YOURS. No platform, no app store, no upsells. About $7 a month, and
everything on it can move to any other server, any time, in one folder.

You will never open a terminal. There are four steps.

## 1. Create the employee (Telegram, 2 minutes)

1. Message @BotFather → `/newbot` → give it a real name (you'll talk to it
   every day)
2. Copy the bot token it gives you
3. Message @userinfobot → copy your numeric user ID

## 2. Prepare the building form (1 minute)

Open [cloud-init.yaml](./cloud-init.yaml) and copy all of it. Replace the
three placeholders — swap the ENTIRE placeholder text, so only your value
remains:

| Placeholder | Replace it with |
|---|---|
| `PASTE-YOUR-BOT-TOKEN-HERE` | the token from BotFather (step 1) |
| `PASTE-YOUR-USER-ID-HERE` | your number from userinfobot — locks the employee to YOU |
| `CHOOSE-A-PASSWORD-HERE` | a password you invent for the one-time setup page |

Before and after, so there's no doubt:

```
before:  TELEGRAM_BOT_TOKEN=PASTE-YOUR-BOT-TOKEN-HERE
after:   TELEGRAM_BOT_TOKEN=8123456789:AAHrX9wkQfEXAMPLEtoken
```

## 3. Rent the office (Hetzner, 5 minutes)

1. hetzner.com → Cloud → create a server
2. Location: nearest you · Image: **Ubuntu 24.04** · Type: the ~€7 shared
   tier (2 vCPU / 8 GB is plenty — the thinking happens elsewhere)
3. Expand **Cloud config** and paste your edited file
4. Create. Wait about 5 minutes while the building assembles itself.

Note: brand-new Hetzner accounts are sometimes asked to verify identity.
If that happens, it usually clears quickly — it's them, not you.

## 4. Connect the brain (browser, 3 minutes)

1. Open `http://YOUR-SERVER-IP:7681` (the IP is on your server's page)
2. Log in — username `setup`, password = the one you chose
3. Follow the friendly wizard: pick **OpenAI Codex** (that's your ChatGPT
   subscription — no API keys, no extra bills), open the link it shows,
   sign in, paste the code back
4. The page seals itself forever when it finishes

Then open Telegram, press **Start** on your bot, and say hello.

## What just happened

- Your employee lives at your server, works 24/7, and answers only to
  your Telegram account
- Its entire life — identity, memory, skills — lives in one folder on
  YOUR server (`/root/.hermes`), portable to any machine on earth
- The only doors open to the internet: Telegram (outbound), SSH (yours),
  and the setup page — which no longer exists

Next: install the blueprint and put it to work —

```
/skills install github.com/lukesbrave/digital-home-blueprint
```

then say: **start the mission**
