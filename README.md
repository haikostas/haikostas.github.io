# Total Reach — social views counter

Counts your total views across YouTube, Instagram, Facebook, TikTok and
Pinterest, and shows the running total on a full-screen display.

A small Python **collector** owns every platform integration (auth, token
refresh, polling, caching) and writes one `data.json`. A separate **display**
page just reads that file and animates the numbers. The two never touch each
other's concerns — so when TikTok inevitably needs a token refresh, the other
four keep showing without a hiccup.

## What works today

| Platform   | Status                                                        |
|------------|---------------------------------------------------------------|
| YouTube    | ✅ Working — free Data API, just needs an API key + channel ID |
| Instagram  | ⏳ Ready to activate — pending Meta App Review                  |
| Facebook   | ⏳ Ready to activate — shares the Meta app with Instagram       |
| TikTok     | 🔑 Code ready — run tiktok_login.py once approved (see below)   |
| Pinterest  | ⏳ Ready to activate — pending Pinterest Standard access        |

The four "pending" platforms show as such on the display without breaking
anything. Each file in `platforms/` has full setup notes for when its access
is approved.

## Quick start (laptop, no Raspberry Pi needed)

```bash
pip install -r requirements.txt

# 1) See the display look its best with fake data:
python collector.py --demo
#    -> open http://localhost:8000

# 2) Go real:
cp config.example.json config.json
#    edit config.json — add your YouTube api_key + channel_id (see platforms/youtube.py)
python collector.py
#    -> open http://localhost:8000
```

`--demo` invents numbers so you can admire the UI before any API is set up.
`--once` polls a single time and exits (handy for testing a new credential).
`--port 8800` changes the web port.

## Two things to start NOW (they're the real bottleneck)

The code is the fast part; **approvals are the slow part**. Kick these off today
so they're ready by the time the code is:

1. Register a **TikTok** developer app (slowest approval).
2. Request Pinterest **Standard access** (Trial tokens expire every 24h and
   won't work for an always-on display).

Meta (Facebook + Instagram) is one app with App Review for the insights
permissions — start that whenever you're ready.

## TikTok login (one-time)

Once your TikTok app's credentials exist (even before full App Review, using a
sandbox / target user), authorize your account once:

1. In config.json, fill `tiktok.client_key`, `tiktok.client_secret`, and
   `tiktok.redirect_uri` (must EXACTLY match a redirect URI registered in the
   TikTok portal, e.g. `http://127.0.0.1:8421/callback`).
2. Run `python tiktok_login.py` — your browser opens TikTok's auth screen,
   you approve, and the token is saved to `tiktok_token.json`.
3. Set `tiktok.enabled = true` and run the collector. It refreshes the token
   automatically from then on.

**Demo video for App Review:** screen-record step 2 from start to finish —
running the script, the browser opening, the TikTok authorization screen, you
approving, the "authorized" page, and the token being saved (and ideally the
collector then showing your TikTok number). That recording is the end-to-end
flow TikTok asks you to upload.

## Adding a platform once it's approved

1. Fill its credentials into `config.json` and set `"enabled": true`.
2. Implement `fetch()` in the matching `platforms/*.py` (the setup notes and a
   call sketch are already in each file).
3. Restart the collector. The display picks it up automatically.

## Moving to the Raspberry Pi later

It's a straight copy — same code, same commands. The extra Pi-only steps will
be: run the collector on boot (a systemd service), and launch Chromium in
kiosk mode pointed at `http://localhost:8000` so it fills the screen. Ask when
you get there and we'll set that up.

## Tuning

`poll_interval_seconds` in config.json (default 1800 = 30 min). View counts
move slowly, so polling often buys nothing and just burns rate limits.
