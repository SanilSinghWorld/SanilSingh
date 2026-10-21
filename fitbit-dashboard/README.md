# My Health Dashboard (Fitbit, no Premium)

A private, single-page dashboard for my Fitbit Inspire 3 data. It reads a Google Takeout export **in the browser**. No server, no account, no upload. The chart and zip libraries are vendored in `vendor/`, so the page makes no network requests.

## What it shows

- **Last 7 days vs. my 28-day baseline:** steps, resting heart rate, time asleep, HRV, sleep score, SpO₂
- **Trends (30 days / 90 days / 1 year / all):** steps with a 7-day average, resting HR, sleep stages, sleep score, HRV, SpO₂, active minutes, weight
- A table view under every chart with the exact values

It does not recreate Premium's proprietary scores (Daily Readiness, Stress Management). The baseline comparison is my own simple average, not Fitbit's algorithm.

## 1. Get the data (free)

1. Go to <https://takeout.google.com> and sign in with the Google account my Fitbit is linked to.
2. Click **Deselect all**, then check **Fitbit** only.
3. **Next step** → export once, `.zip`, a large file size (e.g. 4 GB or 50 GB) → **Create export**.
4. Google emails a download link (minutes to a day or two). Download every part.

## 2. Open the dashboard

- **Computer (easiest):** open `index.html` in Chrome, Safari, Edge or Firefox. Click **Open Takeout .zip or files** and select all the zip parts at once, or use **Open unzipped folder**.
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
