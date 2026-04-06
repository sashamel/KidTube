# KidTube — Setup & Deployment Guide

A safe YouTube wrapper for young children. Hosted as a static web app on Netlify, installed to an Android tablet via "Add to Home Screen".

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire app |
| `config.json` | All settings: PIN, channels, keywords |
| `manifest.json` | PWA metadata (install icon, theme colour) |
| `netlify.toml` | Netlify routing config |

---

## Step 1 — Get a YouTube Data API Key

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (e.g. "KidTube")
3. Go to **APIs & Services → Enable APIs** → search for **YouTube Data API v3** → Enable
4. Go to **APIs & Services → Credentials → Create Credentials → API Key**
5. Click the key → under **API restrictions**, select "YouTube Data API v3"
6. Under **Website restrictions**, add your Netlify domain (e.g. `https://your-app.netlify.app/*`) once you have it
7. Copy the key

Open `index.html` and replace:
```
const API_KEY = 'YOUR_YOUTUBE_API_KEY_HERE';
```
with your actual key.

---

## Step 2 — Put it on GitHub

1. Create a new **public** GitHub repository (e.g. `kidtube`)
2. Upload all four files: `index.html`, `config.json`, `manifest.json`, `netlify.toml`
3. That's your repo — you can edit `config.json` here any time from your PC

---

## Step 3 — Deploy on Netlify

1. Go to [netlify.com](https://netlify.com) and sign up (you can use your GitHub login, or a fresh email)
2. Click **Add new site → Import an existing project → GitHub**
3. Authorize Netlify to access your repo, then select it
4. Leave all build settings blank (no build command, no publish directory — it's a static site)
5. Click **Deploy site**

Netlify will give you a URL like `https://your-app-name.netlify.app`. Every time you push a change to GitHub (including edits to `config.json`), Netlify redeploys automatically within ~30 seconds.

---

## Step 4 — Install on Android Tablet

1. Open Chrome on the tablet
2. Navigate to your Netlify URL
3. Tap the **three-dot menu (⋮)** → **Add to Home screen**
4. Name it "KidTube" and tap Add
5. It now appears as an app icon on the home screen — tap it and it opens fullscreen with no browser chrome

> **Important:** Make sure you're NOT signed into any Google account in that Chrome browser, or sign into a separate child Google account, to avoid the kid's viewing activity appearing in your YouTube recommendations.

---

## Editing the Config

`config.json` controls everything. Edit it in GitHub (click the file → pencil icon) and commit. The app fetches it fresh on each load.

```json
{
  "pin": "2580",
  "channels": [
    { "name": "3 Кота", "id": "UC_T5XbinlhZfQ9reWLAGjxw" }
  ],
  "keywords": {
    "allowlist": [],
    "blocklist": ["scary", "horror"]
  },
  "blockShorts": true
}
```

- **`pin`** — 4-digit PIN to access parent settings and report a video
- **`channels`** — list of channels shown on the home screen
- **`keywords.allowlist`** — for Search only: a result must contain at least one of these words in the title. Leave empty `[]` to allow all.
- **`keywords.blocklist`** — for Search only: results containing these words are hidden
- **`blockShorts`** — set to `true` to hide YouTube Shorts everywhere

To add a channel: find its ID (YouTube channel page → About → Share → Copy channel ID), add it to the list.

---

## Optional: Google Sheet Blacklist (so reported videos are visible from your PC)

When you tap 🚩 Report on a video (PIN-protected), the video ID is stored in the tablet's browser localStorage. Optionally, you can also log it to a Google Sheet so you can see and manage it from your PC.

### Setup (takes ~10 minutes):

1. Create a new Google Sheet
2. Name the first sheet "Blacklist"
3. Add headers in row 1: `A1 = Timestamp`, `B1 = VideoID`, `C1 = Title`
4. Go to **Extensions → Apps Script**
5. Paste this code:

```javascript
function doGet(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Blacklist');
  const videoId = e.parameter.videoId || '';
  const title = e.parameter.title || '';
  if (videoId) {
    sheet.appendRow([new Date().toISOString(), videoId, title]);
  }
  return ContentService.createTextOutput('OK');
}
```

6. Click **Deploy → New deployment → Web app**
7. Set "Who has access" to **Anyone**
8. Copy the web app URL

9. In `index.html`, replace:
```javascript
const BLACKLIST_SHEET_URL = '';
```
with:
```javascript
const BLACKLIST_SHEET_URL = 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec';
```

Now every reported video also logs to your Google Sheet. Periodically, you can copy those video IDs into `config.json` as a permanent blocklist if you want them blocked even after clearing the app's local storage. (There's no built-in blocklist in config.json currently — you'd add it as `"blockedVideoIds": ["abc123", ...]` and add handling in the code if needed.)

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Home screen shows "Could not load config.json" | Make sure config.json is in the root of your repo and deployed |
| Videos don't load | Check API key is set in index.html and YouTube Data API v3 is enabled |
| API quota exceeded | You get 10,000 units/day free. Each channel load costs ~12–15 units. Should be fine for daily use. |
| Video plays with YouTube recommendations visible | The embed uses `rel=0` to minimise this. YouTube may still show same-channel suggestions at the end — this is a YouTube limitation on embeds. |
| "Add to Home Screen" not appearing | Make sure you're in Chrome (not Firefox or Samsung Browser) and navigating to the https:// URL |

---

## Blocking Ads with DNS filtering

YouTube's embedded player shows ads on monetised videos. The cleanest free solution is to set the tablet's DNS to a service that blocks ad-serving domains at the network level, before they even reach the browser.

### Recommended: AdGuard DNS (free, no account needed)

1. On the tablet, go to **Settings → Wi-Fi**
2. Long-press your Wi-Fi network → **Modify network** (or tap the ⚙️ gear icon)
3. Set **IP settings** to **Static** (so the DNS setting sticks)
4. Set **DNS 1** to `94.140.14.14`
5. Set **DNS 2** to `94.140.15.15`
6. Save

That's it. AdGuard DNS blocks ad and tracking domains for everything on that Wi-Fi network — YouTube ads, in-app ads, website ads.

**Alternative: NextDNS** (free tier: 300,000 queries/month, more control)
- Go to [nextdns.io](https://nextdns.io), create a free account
- You get a custom DNS address and a dashboard where you can see/block specific domains
- Set it the same way as above

### For near-100% ad blocking: Firefox + uBlock Origin

Chrome on Android doesn't support extensions, but Firefox does:
1. Install **Firefox for Android** from the Play Store
2. Open Firefox → tap the three-dot menu → **Add-ons** → install **uBlock Origin**
3. Navigate to your KidTube URL in Firefox
4. Tap the three-dot menu → **Add to Home Screen**

The KidTube app icon will now launch from Firefox instead of Chrome — it looks and works identically, but uBlock Origin silently removes ads before they render.

### YouTube Premium

If you have (or are considering) YouTube Premium, it removes ads everywhere including embedded players. Just sign the tablet into that Google account in Firefox or Chrome — but remember to use a separate browser profile or incognito mode to avoid mixing watch history with your own account.
