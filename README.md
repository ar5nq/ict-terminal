# ICT Terminal — installable app + 24/7 phone alerts

Two parts that work together:

1. **The terminal as an app.** `index.html` (your full SCAN/QR/charts/workspace
   terminal) hosted on GitHub Pages, installable to your phone's home screen
   like a real app — its own icon, opens full-screen, no browser chrome.
2. **A 24/7 background scanner** (`scan.js`, runs on GitHub's servers on a
   schedule) that pushes a real notification straight to that installed app —
   XAUUSD/NAS100/EURUSD only — the moment one crosses into STRONG. Your
   phone/laptop don't need to be open, awake, or even on.

## What it does NOT do

- **No manual price-level alerts on the background scanner.** Those still
  live only in the app's own local storage — this covers the automatic SCAN
  STRONG-crossover alerts on your 3-pair watchlist.
- **Not a native App Store app.** It's a PWA (Progressive Web App) — installs
  and behaves like an app (icon, full-screen, push notifications) but isn't
  distributed through the App Store/Play Store. No developer account needed,
  which is also why this is buildable at all without one.
- **No sub-15-minute precision.** GitHub's free scheduler has a 5-minute
  floor and no exact-time guarantee.

## Setup

### 1. Create the repo and host it
- Create a new **public** GitHub repo (public = unlimited free Actions
  minutes + free Pages hosting).
- Upload every file in this folder, preserving the folder structure
  (`icons/`, `.github/workflows/`, etc).
- **Settings → Pages → Source: Deploy from a branch → Branch: `main`, folder
  `/ (root)` → Save.** Wait a minute or two, then note the URL GitHub gives
  you — something like `https://yourname.github.io/ict-scan-alerts/`.

### 2. Install it on your phone
- Open that Pages URL **on your phone**, in Safari (iOS) or Chrome (Android).
- **iOS:** tap the Share icon → **Add to Home Screen**.
- **Android (Chrome):** tap the menu (⋮) → **Install app** (or you'll get an
  automatic install banner).
- Open the app from the **home screen icon** you just created, not the
  browser tab — this matters, especially on iOS, for push to work reliably.

### 3. Turn on push notifications in the app
- In the app, go to the **ALERT** panel → scroll to **PHONE PUSH** → tap
  **ENABLE PHONE ALERTS** → allow notifications when prompted.
- A JSON block appears with a **COPY** button. Copy it — this is this
  specific device's push address, and it's the one-time link between "your
  phone" and "the background scanner."

### 4. Add the secrets
In the repo: **Settings → Secrets and variables → Actions → New repository
secret**, add all four:

| Secret name | Value |
|---|---|
| `PUSH_SUBSCRIPTION` | the JSON you copied in step 3 |
| `VAPID_PUBLIC_KEY` | `BJR2pCXHk-54LcdJTaDeQO5rgnCmQZOmsizokSGQIIU_jdCA8iWDIzl2BttDpBwh1T94OoRRjtZSRVUNLJw7H_Y` |
| `VAPID_PRIVATE_KEY` | `ePV1NvCAkXY2HkEv0b_BqXgns2jSBT_Fafq6j6x9ZDM` |
| `VAPID_SUBJECT` | `mailto:you@example.com` (any email of yours — optional, has a fallback) |

The public/private pair above is a matched set generated for this project —
they only work with each other. `VAPID_PUBLIC_KEY` is also already embedded
in `index.html` to match; if you ever regenerate your own pair (`npx
web-push generate-vapid-keys`), update **both** the secret and the constant
near the bottom of `index.html`, or push and the app will subscribe with a
key the server doesn't recognize.

### 5. Turn it on
- **Actions** tab → enable workflows if prompted → click into **ICT SCAN
  alerts** → **Run workflow** to trigger it once manually.
- Check the run log: it'll print each instrument's current score/label, and
  either "web push sent" (if something's STRONG) or nothing (if not — that's
  normal, it only pushes on an actual crossover). Either way, a clean log
  with no errors means the plumbing works.
- After that it runs on its own every ~15 minutes.

## How state works

`state.json` remembers each watchlist instrument's last label so it only
notifies you the moment something *enters* STRONG, not every run while it
stays there. The workflow commits this back to the repo each run, which also
keeps GitHub from disabling the schedule for inactivity.

## Re-subscribing

If you reinstall the app, clear site data, or switch phones, the old
`PUSH_SUBSCRIPTION` stops working silently. Repeat steps 3–4 to reconnect.

## Known reliability risks

- **Yahoo Finance's chart endpoint** is free and unofficial — it can
  occasionally rate-limit or block automated requests. If runs start failing
  in the Actions log, this is almost always the cause, not the code.
- **iOS push for home-screen web apps** needs iOS 16.4+, and Apple's
  implementation has historically been less consistent than Android's. If
  notifications stop arriving, first check the app is still installed as a
  home-screen icon (not just bookmarked) and re-run step 3.
- **`ntfy.sh` fallback:** if you also set an `NTFY_TOPIC` secret (see the
  ntfy.sh app), alerts go out on both channels — useful as a backup while
  you confirm Web Push is working.
