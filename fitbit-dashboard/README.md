# My Health Dashboard (Fitbit, no Premium)

A private, single-page dashboard for my Fitbit Inspire 3 data. A **Sync from Fitbit** button pulls the latest data from the Google Health API, and a Google Takeout export can be loaded for full history. Everything runs in the browser: data goes straight from Google to the page, with no server in between. Chart and zip libraries are vendored in `vendor/`. The only outside script is Google's sign-in library.

The band syncs to the Fitbit app over Bluetooth, and the app syncs to Google's cloud. The button reads from the cloud, so data is as fresh as the Fitbit app's last sync. Open the Fitbit app first to force one.

## What it shows

- **Last 7 days vs. my 28-day baseline:** steps, resting heart rate, time asleep, HRV, sleep score, SpO₂
- **Trends (30 days / 90 days / 1 year / all):** steps with a 7-day average, resting HR, sleep stages, sleep score, HRV, SpO₂, active minutes, weight
- A table view under every chart with the exact values

It does not recreate Premium's proprietary scores (Daily Readiness, Stress Management). The baseline comparison is my own simple average, not Fitbit's algorithm.

## Sync button: one-time setup (about 15 minutes)

The legacy Fitbit Web API is being shut down (Google's notices point to late October 2026), so this uses its replacement, the **Google Health API**. You create your own private Google Cloud app that only you can use.

1. **Host the page.** In the GitHub repo: Settings → Pages → Deploy from a branch → pick the branch with `fitbit-dashboard/` → `/ (root)`. The dashboard is then at `https://sanilsinghworld.github.io/SanilSingh/fitbit-dashboard/`. The code is public, but your data never touches GitHub.
2. Go to <https://console.cloud.google.com/projectcreate> and create a project (e.g. `my-health-dashboard`).
3. Enable the API: <https://console.cloud.google.com/apis/api/health.googleapis.com> → **Enable**.
4. **Google Auth Platform → Branding:** app name, your email as support and developer contact.
5. **Audience:** User type **External**, leave it in **Testing**, and add the Google account your Fitbit is linked to as a **test user**.
6. **Data Access → Add or remove scopes:** add these three read-only scopes:
   - `https://www.googleapis.com/auth/googlehealth.activity_and_fitness.readonly`
   - `https://www.googleapis.com/auth/googlehealth.health_metrics_and_measurements.readonly`
   - `https://www.googleapis.com/auth/googlehealth.sleep.readonly`
7. **Clients → Create client → Web application.** Under *Authorized JavaScript origins* add `https://sanilsinghworld.github.io`. No redirect URI is needed.
8. Copy the **Client ID** (ends in `.apps.googleusercontent.com`), open the dashboard → **Sync settings** → paste → **Save**.
9. Tap **Sync from Fitbit**. Google will warn *"Google hasn't verified this app"*. That's expected for your own testing-mode app: **Continue**, tick all the boxes, allow.

How it behaves:
- The first sync pulls the last 90 days. Later syncs fetch from your last sync onward, plus a few days back so late sleep and HRV data fill in.
- The sign-in token stays in memory only and lasts about an hour. After that, the button briefly opens Google sign-in again.
- Synced data is saved in this browser (localStorage) so it's there next time. **Sync settings → Forget my data on this device** deletes it. Any other page you host on `sanilsinghworld.github.io` shares that storage, so don't host pages you don't trust there.
- On iPhone, use it in Safari, or add it to the Home Screen from Safari's share menu. If the sign-in pop-up is blocked, allow pop-ups for the site.
- Sync covers steps, active minutes, resting HR, HRV, SpO₂, sleep stages and weight (in kg). Sleep score isn't in the Google Health API, so it only comes from Takeout.

## Takeout: full history (free)

1. Go to <https://takeout.google.com> and sign in with the Google account my Fitbit is linked to.
2. Click **Deselect all**, then check **Fitbit** only.
3. **Next step** → export once, `.zip`, a large file size (e.g. 4 GB or 50 GB) → **Create export**.
4. Google emails a download link (minutes to a day or two). Download every part.

## Loading a Takeout export

Takeout data is merged with synced data and saved in the browser the same way.

- **Computer (easiest):** open the dashboard (or a local copy of `index.html`) in Chrome, Safari, Edge or Firefox. Click **Open Takeout .zip or files** and select all the zip parts at once, or use **Open unzipped folder**.
- **iPhone:** save the zip to the Files app, open the dashboard in Safari, and pick the zip with **Open Takeout .zip or files**. Very large exports can be slow on a phone; a computer handles them better.
- **Try with demo data** shows the layout before the export arrives.

Minute-level heart rate, calories and distance files are skipped on purpose. They can be gigabytes, and daily resting HR, steps and active minutes already cover the useful trends.

## Files it reads

| Data | Takeout file |
|---|---|
| Steps | `Global Export Data/steps-*.json` |
| Resting HR | `Global Export Data/resting_heart_rate-*.json` |
| Sleep stages | `Global Export Data/sleep-*.json` |
| Active minutes | `Global Export Data/{very,moderately,lightly}_active_minutes-*.json` |
| Weight | `Global Export Data/weight-*.json` |
| HRV | `Daily Heart Rate Variability Summary*.csv` |
| SpO₂ | `Daily SpO2*.csv` |
| Sleep score | `sleep_score.csv` |

If Google changes the export layout, the **About this data** section lists every JSON/CSV file it didn't recognise, so the patterns at the top of the script in `index.html` can be updated.

Never commit the export itself. `.gitignore` blocks zip, JSON and CSV files in this folder.
