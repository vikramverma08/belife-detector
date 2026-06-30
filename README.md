# BeLife Multi-Center Duplicate Detector

A single self-contained web app that detects duplicate / re-uploaded diagnostic
reports per center, using each center's own Google Drive folder as its database.
No server — everything runs in the browser.

## Deploy on Vercel

1. Push this folder to a GitHub repository.
2. On [vercel.com](https://vercel.com), click **Add New → Project**, import this repo.
3. No build settings needed — it's a static site. Click **Deploy**.
4. Copy your live URL, e.g. `https://your-app.vercel.app`.

## ⚠️ Required: add your Vercel URL to Google OAuth

Google sign-in will fail until you do this:

1. Go to **Google Cloud Console → APIs & Services → Credentials**.
2. Open your **OAuth Client ID**.
3. Under **Authorized JavaScript origins**, add your exact Vercel URL
   (no trailing slash), e.g. `https://your-app.vercel.app`.
4. Save and wait a few minutes.

Make sure the **Google Drive API** and **Google Picker API** are enabled, and
that you have an **API key** as well. Paste the Client ID + API key into the
app's one-time setup panel.

## Files

- `index.html` — the app (renamed from `belife-multidc-detector.html` so Vercel serves it at the root).
- `GUIDE.md` — full usage guide (setup, Drive folder layout, daily workflow).

See `GUIDE.md` for how to organize each center's folder and the daily workflow.
