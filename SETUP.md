# Great Clips Review Monitor — Setup Guide

**What it does:** Every morning you get an email with your salon's Google review activity — new reviews with their text and rating, your exact current average, and how many more 5★ reviews you need to hit your target. If a low rating (1★ or 2★) appears, you get an instant alert within the hour, not the next day. Works for any number of salons from one setup.

**Cost:** $0/month. Google gives $200 free API credit monthly; this costs under $1.

**Time to set up:** ~30 minutes, one time only.

---

## Before you start — collect these things

For **each salon** you want to monitor:
- The salon's Google Maps listing (search it on maps.google.com to confirm the right one)
- The star breakdown (1★, 2★, 3★, 4★, 5★ counts — tap each bar on the Google Maps rating chart to see exact numbers)

For yourself:
- A Gmail account to send the emails from
- The email address(es) where you want to receive alerts

---

## Step 1 — Get a Google Places API key

1. Go to [console.cloud.google.com](https://console.cloud.google.com) → sign in with any Google account.
2. Create a project (name it anything, e.g. "GC Review Monitor").
3. Add a payment method when prompted — you **will not be charged** (well under the $200 free monthly credit).
4. Search **"Places API"** in the top bar → click it → **Enable**.
5. Go to **Credentials** → **Create Credentials** → **API key** → copy the key.
6. Keep it safe — treat it like a password. Do not paste it in email or chat.

## Step 2 — Find your Place ID(s)

For each salon:
1. Go to [developers.google.com/maps/documentation/places/web-service/place-id](https://developers.google.com/maps/documentation/places/web-service/place-id)
2. Search your salon by full name and address (e.g. "Great Clips, 123 Main St, City").
3. Click the result and confirm the address shown is correct.
4. Copy the Place ID (starts with `ChIJ...`).

## Step 3 — Gmail app password

1. Go to [myaccount.google.com](https://myaccount.google.com) → **Security**.
2. Turn on **2-Step Verification** if not already on.
3. Search **"App passwords"** → create one named "GC Review Monitor" → copy the 16-character code.

## Step 4 — Create a GitHub account and fork this repo

1. Go to [github.com](https://github.com) → Sign up (free).
2. Go to the shared repo link (provided separately) → click **Fork** (top right) → **Create fork**.
3. You now have your own private copy of all the code.

## Step 5 — Configure your salons

In your forked repo, click on `salons.json` → pencil icon to edit.

**Single salon:**
```json
[
  {
    "id": "YOUR_SALON_NUMBER",
    "name": "Great Clips Salon XXXX — Your Location",
    "place_id": "ChIJ_YOUR_PLACE_ID",
    "email_to": "you@email.com, manager@email.com",
    "target_rating": 4.9
  }
]
```

**Multiple salons — just add more entries:**
```json
[
  {
    "id": "1001",
    "name": "Great Clips Salon 1001 — Main Street",
    "place_id": "ChIJ_SALON_1_PLACE_ID",
    "email_to": "you@email.com",
    "target_rating": 4.9
  },
  {
    "id": "1002",
    "name": "Great Clips Salon 1002 — Oak Avenue",
    "place_id": "ChIJ_SALON_2_PLACE_ID",
    "email_to": "you@email.com, salon2manager@email.com",
    "target_rating": 4.8
  }
]
```

Note: each salon can have **different email recipients** and a **different target rating**. Commit changes when done.

## Step 6 — Seed your baseline review counts

For each salon, you need to create a state file with the current star breakdown so the math is exact from day one.

**Easiest way — paste your counts into any Claude conversation and ask:**
> "Create a state_XXXX.json file for my salon. Here are the star counts: 1★=15, 2★=4, 3★=4, 4★=25, 5★=572. Total=620."

Claude will generate the correctly formatted file instantly.

**Or do it manually:** Create a file named `state_YOURSALONID.json` (e.g. `state_4542.json`) with this content:
```json
{
  "total": TOTAL_REVIEWS,
  "sum": SUM_OF_ALL_RATINGS,
  "breakdown": {"1": X, "2": X, "3": X, "4": X, "5": X},
  "seen_review_ids": [],
  "google_rating": 0,
  "bootstrapped": false
}
```

To calculate `sum`: multiply each star by its count and add them.
Example: (1×15)+(2×4)+(3×4)+(4×25)+(5×572) = 15+8+12+100+2860 = **2995**

Upload each state file to your repo root.

## Step 7 — Add secrets

Go to your repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**.

Add these three (they apply to all your salons):

| Name | Value |
|---|---|
| `GOOGLE_PLACES_API_KEY` | from Step 1 |
| `EMAIL_FROM` | the Gmail address sending the emails |
| `EMAIL_APP_PASSWORD` | the 16-character code from Step 3 |

Note: email recipients per salon are configured in `salons.json`, not here.

## Step 8 — Enable the workflows

Go to the **Actions** tab in your repo.

- If you see a prompt saying "Workflows aren't running" → click **Enable workflows**.
- You'll see two workflows: **Daily Review Check** and **Review Alert (Hourly)**.
- Click each one → **Run workflow** → **Run workflow** to test.

Check your inbox. The first daily email will sync your baseline silently. From the next run, you'll get real day-over-day updates. The hourly alert only sends when something new appears.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Wrong salon numbers in email | Check Place ID in salons.json — search more specifically on the Place ID finder |
| Workflow shows red X | Click into it, copy the error, paste into Claude |
| Email never arrives | Check spam folder; confirm EMAIL_FROM and EMAIL_APP_PASSWORD secrets match |
| Math seems off | Paste your star breakdown + the email output into Claude and ask it to verify |
| Want to add a new salon | Add an entry to salons.json + create the matching state_ID.json file |
| Want to pause one salon | Remove or comment out its entry in salons.json |

## Getting help

Upload this guide and any of the code files into a Claude conversation (claude.ai, free account works) and describe what step you're on or what error you're seeing. Claude can read screenshots too — if something looks wrong on screen, just paste the screenshot and ask "what do I do here?"
