# FreqNoisePlotter.html — Setup Guide

One HTML file. No server. No Python. Opens directly from SharePoint in any browser.

---

## How it works

- Runs entirely in the browser
- Signs in with the user's existing Microsoft / work account (MSAL popup)
- Calls the Microsoft Graph API to list files in your SharePoint folder
- Parses and plots the selected files with Plotly.js
- Downloads the plot as a high-res PNG directly to the user's Downloads folder

---

## Step 1 — Azure AD App Registration

You need to do this once. Ask IT if you don't have Azure portal access.

1. Go to https://portal.azure.com
2. Azure Active Directory → App registrations → **New registration**
   - Name: `FreqNoisePlotter` (or anything)
   - Supported account types: **Accounts in this organizational directory only**
   - Redirect URI: choose **Single-page application (SPA)** and enter the full
     URL where the HTML file will live on SharePoint, e.g.:
     `https://contoso.sharepoint.com/sites/QuantumLab/Shared%20Documents/Apps/FreqNoisePlotter.html`
     *(You can add more redirect URIs later if needed)*
3. Click **Register**
4. Copy the **Application (client) ID** and **Directory (tenant) ID** from the Overview page

5. Go to **API permissions → Add a permission → Microsoft Graph → Delegated**
   - Add: `Files.Read.All`
   - Add: `Sites.Read.All`
   - Click **Grant admin consent for [your org]**

> **Important:** Use **Delegated** permissions (not Application), and SPA redirect URI.
> This lets users sign in with their own accounts — no client secret needed.

---

## Step 2 — Edit the HTML file

Open `FreqNoisePlotter.html` in a text editor and fill in the `CONFIG` block near the top:

```javascript
const CONFIG = {
  clientId:   "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",   // from Step 1
  tenantId:   "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",   // from Step 1

  siteUrl:    "https://contoso.sharepoint.com/sites/QuantumLab",

  // Default folder shown on load — user can type a different path in the UI
  folderPath: "/sites/QuantumLab/Shared Documents/ExperimentData",
};
```

---

## Step 3 — Upload to SharePoint

1. Upload `FreqNoisePlotter.html` to any SharePoint document library
   (e.g. `Shared Documents/Apps/`)
2. Right-click the file → **Copy link** to get the direct URL

> SharePoint serves `.html` files directly when opened — no special configuration needed.

---

## Step 4 — Share the link

Paste the SharePoint URL anywhere:
- Confluence page (as a hyperlink or iframe macro)
- Teams message
- Email
- Bookmarks bar

Users click the link, sign in with their Microsoft account (one-time per session),
and the app is ready.

---

## Fixed axis limits (hardcoded in the HTML)

```
X: 1 Hz  →  100 MHz   (1 to 1×10⁸)
Y: 10⁻¹  →  10¹¹  Hz²/Hz
```

These are constants at the top of the script — they never change regardless of data.

---

## File format

The parser expects tab-separated files with a metadata header block, matching
the OE Waves / Measurement Lab phase noise analyzer output format:

- Skips all non-numeric header rows automatically
- Column 1: Frequency (Hz)
- Column 4: Frequency Noise (Hz²/Hz)  ← plotted
- Other columns ignored

It also falls back gracefully to any tab-separated file where col 1 and col 4
are numeric.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Sign-in popup blocked | Allow popups for sharepoint.com in browser settings |
| `redirect_uri_mismatch` error | Add the exact page URL as a SPA redirect URI in Azure |
| 403 from Graph API | Admin consent not yet granted — ask IT to approve the permissions |
| Empty file list | Check the folder path in the UI — must be server-relative |
| Files load but plot is blank | File may not match expected format; check browser console (F12) |
