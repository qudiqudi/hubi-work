# Hubi Time Tracker

A time tracker for the browser, with a cat. You start a session, record your breaks, and stop the session. Hubi the cat walks on the screen during this time. The app is a PWA, operates without a network connection, and keeps your sessions on your device.

Live at https://hubi.work.

## What it does

- Each session keeps the work time, the break time, and one of eight task categories.
- The `History` tab shows sessions for a date range. You can change or delete a session.
- The `Stats` tab shows totals for the week, the month, or all time, and the time for each task.
- You can get all sessions as a CSV file, and restore sessions from the last seven daily snapshots.
- The interface is in English or German.

## How sync works

Sync is optional. The app makes a phrase of 12 different words from a list of 256, and you type it on your second device. The browser encrypts all sessions before each upload. Sync sends only a channel name and encrypted data, not the phrase.

```
cipher   AES-256-GCM (Web Crypto API), random 12-byte IV
key      PBKDF2-SHA-256 from the phrase, 100,000 iterations, random 16-byte salt
channel  SHA-256 of "channel:" + phrase
```

The client sends the data to `https://sync.hubi.work`. The relay in `worker/` is a Cloudflare Worker that keeps one encrypted block for each channel in Workers KV for 30 days. If two devices change the same session, the newer change stays.

Without the server, one device shows a QR code. The second device scans the code with `BarcodeDetector`, or you copy the data manually.

The page also tells the Worker which page of the app you open. The Worker keeps daily counts, with a short hash of IP address and date to count users. Grafana can read the counts with an API key.

## Running it locally

Start a web server in the repository folder:

```
python3 -m http.server 8000 --bind 127.0.0.1
```

Then open http://localhost:8000. The Worker accepts browser requests only from `https://hubi.work` and from `localhost` on port 8000. A local copy uses the live Worker for sync.

The service worker uses its cache first. If you do not see a change, delete the browser data for the page.

## Deployment

Each push to `main` starts `.github/workflows/deploy.yml`. One job publishes the app to GitHub Pages with a new cache name in `sw.js`. A second job deploys the Worker with `wrangler`.

## Stack

HTML, CSS, and JavaScript. No framework, no `npm` package, no build step. The only external code is `qrcodegen.js` from Project Nayuki (MIT license). All data is in `localStorage`. See `CLAUDE.md` for the file list.
