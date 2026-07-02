# Titan Capital Portfolio Pulse — Design Spec

## Purpose

A free, static Next.js dashboard tracking public signals across a shortlist of Titan Capital's portfolio companies. Built as an interview showcase project — a working artifact, not just a pitch. This is a direct replication of the Elevation Portfolio Pulse project (see `vc-portfolio-pulse-workflow` memory), reusing its architecture, data pipeline, and UI approach, adapted to Titan Capital's real portfolio and brand.

## Scope

Static site only. No backend, no API routes, no database, no auth. Data is fetched by a Node script run manually (or on a schedule) and written to a JSON snapshot that the site reads at build time.

## Companies (10)

Selected for having both public news coverage and an app-store listing:

1. Razorpay — fintech/payments
2. Ola — mobility
3. Urban Company — home services marketplace
4. Mamaearth — D2C personal care
5. Jupiter — neobank
6. Khatabook — SMB fintech/bookkeeping
7. Bewakoof — fashion D2C
8. Park+ — parking/auto tech
9. Teachmint — edtech
10. Netmeds — online pharmacy

Each company needs: name, founder name(s) for founder-mention search, a news search query, and verified iOS/Android app-store IDs (verified via each scraper's own `.search()` function before hardcoding — unverified IDs were wrong for most companies last time).

## Signals

Same three signals as Elevation Portfolio Pulse:

1. **News mentions** — via Firecrawl Search API (`sources: ['news']`), filtered to the last 90 days. Firecrawl mixes relative ("3 days ago") and absolute ("Sep 25, 2018") date strings with no built-in recency filter, so filtering happens client-side in `fetch-news.ts` (reused as-is).
2. **Founder mentions** — same Firecrawl Search mechanism, queried by founder name.
3. **App-store rating** — iOS via `app-store-scraper`, Android via `google-play-scraper` (imported via `.default` — the CommonJS interop bug from last time), both called with `country: 'in'` to avoid 404s on India-only listings.

## Status Derivation

Reuse `lib/status.ts` unchanged as a starting point:
- `hot`: news count ≥ 15
- `watch`: news count ≥ 6
- `stable`: news count ≤ 5

These thresholds were calibrated against Elevation's real data distribution, not Titan's. Once Titan's real data is fetched, check the distribution and recalibrate if it skews all-one-bucket (same issue hit last time).

## Visual Design

Reuse the same glassmorphism + parallax-scroll treatment, retargeted to Titan Capital's real brand palette (pulled from `titancapital.vc`'s live CSS, not guessed):

- `--color-tc-navy: #172d4f` — primary (dominant color on their site, used for headers/structure)
- `--color-tc-coral: #e1624b` — accent (status highlights, CTAs)
- `--color-tc-blue: #1890d7` — secondary accent
- `--color-tc-cream: #EEEADD` — background tint / glass base

Card structure, animation, and layout components (`ParallaxBackground`, `CompanyCard`, `CompanyDetailView`, `StatusBadge`) are reused with only color-token and copy changes — no structural UI changes planned.

## Data Pipeline & Architecture

Identical to Elevation Portfolio Pulse:
- Next.js App Router, `output: 'export'` (static export).
- `scripts/refresh-data.ts` orchestrates per-signal fetches with per-signal try/catch isolation, writes `data/portfolio.json`.
- `FIRECRAWL_API_KEY` read from `.env` (gitignored), never pasted into chat — user pastes it into the `.env` file directly via their editor.
- New Firecrawl account/key (separate from Elevation's), new GitHub repo, new Vercel project — kept fully separate from the Elevation project per-fund.

## Hosting & Cost

Vercel Hobby tier (free, no card), git-connected auto-deploy. Root Directory must be set correctly at import time — this project's repo root *is* the Next.js app root (unlike Elevation's, where the app lived in a subfolder of a larger repo), so no Root Directory override should be needed this time.

## Error Handling

Same as Elevation: each signal fetch (news, founder mentions, app rating) is wrapped independently so one failing signal (e.g. no verified iOS app for a company) doesn't block the others. Missing signals render as "—" in the UI, distinguished from a genuine zero count.

## Testing

Reuse the existing test suite structure (`tests/companies.test.ts`, `tests/status.test.ts`, `tests/fetch-news.test.ts`, `tests/fetch-app-rank.test.ts`), updated for Titan's company data. Same TDD approach for any new logic (should be minimal — this is primarily a data/config/copy swap, not new features).

## Out of Scope

- No new signals beyond the three above.
- No structural UI/animation changes beyond color retargeting.
- No live/server-rendered data — refreshed snapshot only, same as Elevation.
