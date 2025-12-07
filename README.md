# Foothills-DashboardFoothills-Dashboard
===================

Purpose
- Live situation dashboard centered on Boulder Ridge Rd SE (Monroe, WA 98272).
- Shows: local threat score, NWS emergency alerts, weather, traffic placeholders for US-2 and SR-522, utility outages (SnoPUD, PSE, Xfinity), and nearby crime trend snippets.

Quick start (Option A: local prepare & push)
1. In the repo, create files exactly as provided:
   - index.html
   - config.example.json
   - README.md
2. Copy config.example.json -> config.json and edit values:
   - Fill `openWeatherKey` (optional) and any feed URLs you can obtain.
   - If you want exact WSDOT corridors, put API key and corridor IDs in `wsdot`.
3. Keep config.json out of the public repo if it contains keys:
   - Add `config.json` to .gitignore before committing if you prefer.
4. Commit & push:
   git init
   git add .
   git commit -m "Add Foothills Dashboard"
   git remote add origin https://github.com/dredger55/Foothills-Dashboard.git
   git branch -M main
   git push -u origin main

Feeds & notes
- NWS Alerts: used live (no key) via https://api.weather.gov/alerts/active?point={lat},{lon}
- Weather: NWS forecast used if available; OpenWeatherMap is optional (requires key).
- WSDOT Traffic: WSDOT has an API (https://wsdot.com/traffic/api/) — register for a key and add corridor IDs to config.json. Many WSDOT endpoints block cross-origin requests; if CORS blocks, run the optional proxy (below).
- Utility outages:
  - Snohomish PUD: outage map at https://outagemap.snopud.com/ — many ArcGIS maps expose a GeoJSON FeatureServer query. Put that query URL into config.outages.snoPUDFeed.
  - PSE & Xfinity: public pages vary by region; some endpoints require scraping or a small server-side proxy.
- Crime: replace `crimeFeed` with a Socrata-style open-data endpoint or PD dataset that returns JSON with lat/lon and date fields.

CORS / Proxy
- If any feed blocks browser requests with CORS, run a small proxy on your machine / small server. Example Node/Express proxy is below (save as proxy.js and run with `node proxy.js`), then replace feed URLs with your proxy URL + original feed URL.

Optional Node proxy (example)
```js
// proxy.js (optional)
// Simple CORS proxy for server-side fetching of 3rd-party JSON that blocks browser CORS.
// Usage: node proxy.js
const express = require('express');
const fetch = require('node-fetch');
const app = express();
const PORT = process.env.PORT || 3001;
app.get('/fetch', async (req, res) => {
  try {
    const url = req.query.url;
    if(!url) return res.status(400).send('missing url');
    const resp = await fetch(url);
    const text = await resp.text();
    res.header('Access-Control-Allow-Origin','*');
    res.send(text);
  } catch(e) {
    res.status(500).send('error');
  }
});
app.listen(PORT, ()=>console.log('proxy listening',PORT));
