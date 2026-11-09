# PHISHGUARD — Intelligent Phishing Detection & Monitoring System

> **DETECT. ANALYSE. DEFEND.**
> A Security Operations Centre (SOC) style console for phishing URL detection, threat
> intelligence cross-matching and security event monitoring — built as a BCA final-year
> / placement showcase project.

![status](https://img.shields.io/badge/mode-DEMO-orange) ![frontend](https://img.shields.io/badge/frontend-Next.js_16-black) ![license](https://img.shields.io/badge/use-educational-green)

**⚠️ Honest-build disclosure:** this repository ships with a **demo layer**. The scanner is a
transparent local heuristic rule engine, the intelligence providers are simulated, and every
screen that shows sample data is labelled **DEMO DATA**. No ML model, no external threat API,
and no fake "live" security status is claimed anywhere. Model quality metrics (precision /
recall / F1 / confusion matrix) deliberately render as **UNAVAILABLE** until a real evaluated
model is connected.

---

## ✨ Feature Overview

| Module | Highlights |
| --- | --- |
| **Overview Dashboard** | KPI cards with count-up animation, 24h threat-activity area chart, risk-distribution donut, recent security events, detection-engine health, threat radar and a terminal-style activity feed |
| **Live URL Scanner** | Glowing URL console (paste / clear / validate), cinematic staged pipeline (INITIALIZING → ANALYSING → EXTRACTING FEATURES → CHECKING INTELLIGENCE → GENERATING VERDICT), streaming scan terminal, circular risk gauge (0–100), LOW/MEDIUM/HIGH classification, URL-structure breakdown, weighted detection reasons, recommended action |
| **Threat Intelligence** | Domain/URL lookup with deterministic demo dataset, reputation verdict with confidence bar, 5-provider cross-match incl. honest `ERROR` and `UNAVAILABLE` provider states, provider status catalogue |
| **Scan History** | Searchable/filterable table (URL, scan ID, risk, source), 4-way sort, detail drawer with gauge + reasons, one-click **CSV export** of the filtered set |
| **Detection Analytics** | Phishing-vs-legitimate bars, scan-activity combo chart, risk donut, 24h activity — plus honestly `UNAVAILABLE` precision/recall/F1 and confusion-matrix sections |
| **Security Events** | SIEM-style stream (ID, timestamp, severity, category, source, description), severity filters with live counts, search, CSV export |
| **API Management** | Backend connection panel, real health-check probe with timeout/error handling, engine status grid, planned FastAPI endpoint contract, server-side secret handling (masked, never in browser) |
| **Settings** | Operator profile, scanner sensitivity (STANDARD/STRICT affects rule weights), motion/grid/accent appearance controls, runtime backend-URL override, demo-data reset & reseed |

Everything is responsive (mobile drawer navigation, fluid grids) and respects
`prefers-reduced-motion` plus an in-app animation toggle.

---

## 🧱 Tech Stack

- **React 19 + TypeScript 5** — strict typed components
- **Next.js 16 (App Router)** — single-entry shell with a lightweight hash router (`#/overview`, `#/scanner`, …) giving React-Router-style deep links, refresh persistence and working sidebar navigation
- **Tailwind CSS 4** — custom SOC theme (see palette below)
- **shadcn/ui** — buttons, inputs, sheets, switches, toasts
- **Recharts** — all dashboard/analytics charts
- **Zustand** (persisted) — settings, navigation and scan/event stores
- **Framer Motion** — page transitions, pipeline choreography
- **Lucide Icons** — consistent iconography (no emoji)

### Colour system

| Token | Hex | Usage |
| --- | --- | --- |
| Ink (main bg) | `#050807` | app background |
| Ink-2 | `#0A100D` | sidebar / secondary surfaces |
| Panel (cards) | `#0D1712` | cards & modals |
| Neon green | `#39FF14` | primary accent / low-risk |
| Electric cyan | `#00E5FF` | info / lookups / medium data |
| Muted green | `#1B6B42` | radar rings / hairlines |
| Danger | `#FF304F` | high-risk / critical |
| Warning | `#FFB020` | medium-risk / demo badges |
| Text | `#E8F5EC` / `#84968A` | primary / secondary |

Typography: **Space Grotesk** (display), **Inter** (UI), **JetBrains Mono** (terminal, IDs,
timestamps).

---

## 🚀 Getting Started

```bash
# 1. install
bun install          # or: npm install / pnpm install

# 2. environment (optional — see .env.example)
cp .env.example .env.local

# 3. run
bun run dev          # http://localhost:3000

# 4. quality
bun run lint
```

### Demo URLs to try in the scanner

| Expectation | URL |
| --- | --- |
| 🟢 Low risk | `https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS` |
| 🟠 Medium risk | `https://secure-login.portal-user5821.com/auth` |
| 🔴 High risk | `http://paypal-verify-account.tk/webscr?cmd=_login-run` |
| 🔴 High risk (IP host) | `http://192.168.44.7:8081/secure/update-billing.html` |

---

## 🔌 Backend Integration (Python FastAPI)

The frontend talks to the backend exclusively through **one service layer**:
`src/lib/phishguard/api.ts`.

- Set `NEXT_PUBLIC_PHISHGUARD_API_URL` (build time) **or** use *Settings → Backend
  Integration* (runtime override) and the app automatically switches from the **mock adapter**
  to live HTTP calls — no component changes.
- Without a backend URL the fully-functional **mock adapter** (localStorage persistence +
  deterministic demo datasets + simulated latency) keeps every feature working.

### Planned contract

```
POST /api/v1/scans                 → run a scan        { url, source }
GET  /api/v1/scans                 → list scans
GET  /api/v1/scans/{id}            → single verdict
GET  /api/v1/dashboard/summary     → overview KPIs
GET  /api/v1/dashboard/analytics   → chart series
GET  /api/v1/health                → service health
```

Suggested FastAPI sketch:

```python
# main.py (sketch)
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="PhishGuard API", version="1.0.0")

class ScanIn(BaseModel):
    url: str
    source: str = "API"

@app.post("/api/v1/scans")
async def create_scan(body: ScanIn):
    verdict = run_detection_pipeline(body.url)   # ML + rules
    persist(verdict)
    return verdict

@app.get("/api/v1/health")
async def health():
    return {"status": "ok", "ml_engine": "ready", "feeds": "connected"}
```

> **Secrets policy:** provider API keys (VirusTotal, OpenPhish, URLhaus…) live **only** in the
> backend environment. The browser never sees or transmits keys — API Management displays
> masked placeholders to make this explicit.

---

## 📁 Project Structure

```
src/
├─ app/
│  ├─ layout.tsx            # fonts, metadata, toaster
│  ├─ globals.css           # SOC theme tokens + animations
│  ├─ icon.svg              # favicon (shield-eye mark)
│  └─ page.tsx              # app shell: sidebar + topbar + hash router + transitions
├─ components/
│  ├─ ui/                   # shadcn/ui primitives
│  └─ phishguard/
│     ├─ logo.tsx           # original shield+digital-eye brand mark (inline SVG)
│     ├─ nav-config.ts      # sidebar map + page metadata
│     ├─ primitives.tsx     # cards, badges, states, stat cards
│     ├─ visuals.tsx        # risk gauge, threat radar, scan rings, terminal feed
│     ├─ charts.tsx         # themed Recharts wrappers
│     ├─ sidebar.tsx / topbar.tsx / footer.tsx
│     └─ pages/             # 8 feature pages (overview, scanner, threat-intel, …)
├─ hooks/phishguard/
│  └─ use-phishguard.ts     # hash router, clock, count-up, media query
└─ lib/phishguard/
   ├─ types.ts              # domain types (mirror of backend contract)
   ├─ scanner-engine.ts     # DEMO heuristic rule engine (deterministic)
   ├─ demo-data.ts          # seeded datasets + analytics builders
   ├─ api.ts                # service layer: live adapter + mock adapter
   ├─ store.ts              # zustand stores (UI, scans, events)
   └─ format.ts             # formatting, CSV export, PRNG helpers
```

### How the demo scanner verdicts are produced

`scanner-engine.ts` applies ~15 transparent URL-structure rules (IP hosts, missing TLS,
punycode, high-abuse TLDs, brand impersonation, lure keywords, open-redirect params,
subdomain chains, shorteners, executables, entropy…) with weighted scores, clamped to 0–100:

- `≥ 70` → **HIGH_RISK** · `≥ 30` → **MEDIUM_RISK** · else **LOW_RISK**

Same URL ⇒ same verdict (deterministic). This is a *teaching-grade rule engine*, not a
trained model — and the UI says so on every result.

---

## 🧭 Navigation Map

| Hash route | Page |
| --- | --- |
| `#/overview` | Overview Dashboard (default) |
| `#/scanner` | Live Scanner |
| `#/threat-intel` | Threat Intelligence |
| `#/history` | Scan History |
| `#/analytics` | Detection Analytics |
| `#/events` | Security Events |
| `#/api` | API Management |
| `#/settings` | Settings |

---

## 🎓 Interview Talking Points

1. **Architecture** — single API service layer with adapter pattern (mock ↔ live) demonstrates
   production readiness before the backend exists.
2. **Honesty by design** — demo data labelling, unavailable metrics and provider error states
   reflect real SOC practice: never fake telemetry.
3. **Detection logic** — the weighted rule engine mirrors real anti-phishing heuristics and is
   deterministic and auditable.
4. **UX engineering** — reduced-motion support, keyboard-navigable tables/sheets, ARIA roles,
   CSV export, sticky footers, responsive drawer navigation.
5. **Extensibility** — switching to the FastAPI backend requires one env var and zero UI edits.

---

*PHISHGUARD is an educational project. It does not connect to live threat intelligence and
should not be used as a production security control.*
