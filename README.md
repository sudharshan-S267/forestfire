# Forest Fire Detection

A modern web app that pulls **NASA FIRMS** (Fire Information for Resource Management System) satellite data, applies **AI-based filtering and scoring**, and shows **exact-location fire alerts** on an interactive map.

## Features

- **Fire data** – VIIRS/MODIS (same sensors used by ISRO/FSI); global via NASA FIRMS API
- **Satellite (Earth-style) map** – ESRI World Imagery base layer; optional dark theme
- **India (ISRO/FSI)** – India region preset; official alerts at [FSI Forest Fire](https://fsiforestfire.gov.in/)
- **AI filtering** – Confidence + FRP + brightness scoring; clustering to reduce noise
- **Exact location alerts** – Lat/long for each detection with risk score (0–100)
- **Region & date** – Presets (World, India, Americas, etc.) and 1–30 day range
- **Email alerts** – Subscribe by email + region; get notified when new fires are detected (Resend)
- **Auto-refresh** – Data refreshes every 5 minutes

## Tech Stack

- **Next.js 16** (App Router), **TypeScript**, **Tailwind CSS**
- **Leaflet** + **React-Leaflet** for the map
- **NASA FIRMS API** for fire data
- **date-fns**, **lucide-react** for UI

## Setup

1. **Clone and install**

   ```bash
   cd forest-fire-detection
   npm install
   ```

2. **NASA FIRMS API key (required)**

   - Go to [Request FIRMS Map Key](https://firms.modaps.eosdis.nasa.gov/api/map_key/)
   - Sign up and get your free **MAP_KEY**
   - Create `.env.local` in the project root:

   ```env
   NASA_FIRMS_MAP_KEY=your_key_here
   ```

3. **Run**

   ```bash
   npm run dev
   ```

   Open [http://localhost:3000](http://localhost:3000).

4. **Email alerts (optional)**

   - Sign up at [Resend](https://resend.com) and get an API key.
   - Add to `.env.local`:
     ```env
     RESEND_API_KEY=re_xxxx
     RESEND_FROM_EMAIL=Forest Fire Alerts <onboarding@resend.dev>
     ```
   - Users can subscribe on the app; confirmation and alert emails are sent via Resend.
   - To send alerts on a schedule, call `GET /api/cron/check-fires` (e.g. every 15–60 min). Protect with a secret:
     ```env
     CRON_SECRET=your_secret
     ```
     Then call: `GET /api/cron/check-fires` with header `Authorization: Bearer your_secret`. On Vercel, add a [Cron Job](https://vercel.com/docs/cron-jobs) that hits this URL with the secret.

## API

- **GET `/api/fires`** – Fire data. Query: `bbox`, `dayRange` (1–30), `clusterKm`, `minConfidence`. Returns `{ fires, count, lastUpdated }`.
- **POST `/api/subscribe`** – Subscribe to email alerts. Body: `{ email, bbox?, regionLabel? }`. Returns `{ ok, id, message }`.
- **POST `/api/unsubscribe`** – Unsubscribe. Body: `{ email, id? }`.
- **GET `/api/cron/check-fires`** – Run alert job (send emails for new fires). Requires `Authorization: Bearer CRON_SECRET`.

## Project structure

```
src/
  app/
    api/fires/route.ts       # Fire data
    api/subscribe/route.ts   # Email subscription
    api/unsubscribe/route.ts
    api/cron/check-fires/    # Cron: send alert emails
    page.tsx                 # Dashboard (map + alerts + subscribe)
    layout.tsx, globals.css
  components/
    FireMap.tsx          # Leaflet map with fire markers
    AlertsPanel.tsx      # List of alerts with exact location
  lib/
    nasa-firms.ts         # NASA FIRMS API client
    ai-filter.ts          # Scoring + clustering
    subscriptions.ts     # Email subscription store (data/subscriptions.json)
    email.ts              # Resend: confirmation + alert emails
    types.ts              # FireEvent, etc.
```

## Disclaimer

This app is for **education and awareness** only. Do not rely on it for operational firefighting or emergency decisions. Use official sources (e.g. local fire agencies, NASA FIRMS) for real-world use.
