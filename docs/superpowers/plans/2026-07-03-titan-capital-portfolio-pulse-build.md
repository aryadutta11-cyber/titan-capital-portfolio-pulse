# Titan Capital Portfolio Pulse Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Retarget the existing Elevation Portfolio Pulse scaffold (already cloned into this repo) into Titan Capital Portfolio Pulse — swap the 10 tracked companies, retarget the brand palette and copy, and leave the data pipeline, UI structure, and tests untouched per the design spec.

**Architecture:** This is a data/config/copy swap on top of an already-working Next.js static-export app. No new components, no new signals, no structural UI changes. Three areas change: (1) `data/companies.ts` gets Titan's 10 portfolio companies with verified app-store IDs, (2) `app/globals.css` color tokens get renamed from `ec-*` (Elevation) to `tc-*` (Titan) with new hex values, (3) every Tailwind class referencing an `ec-*` token across `app/` and `components/` gets renamed to the matching `tc-*` token. Copy strings ("Elevation Capital" → "Titan Capital") and `package.json`/`README.md`/`app/layout.tsx` metadata get updated to match.

**Tech Stack:** Next.js 16 (App Router, static export), React 19, Tailwind CSS 4 (`@theme inline` tokens), Framer Motion, Vitest, TypeScript, `@mendable/firecrawl-js`, `app-store-scraper`, `google-play-scraper`.

## Global Constraints

- Static site only — no backend, no API routes, no database, no auth (spec: Scope).
- Reuse `lib/status.ts` thresholds unchanged for now: `hot` ≥ 15 news, `watch` ≥ 6, `stable` ≤ 5 (spec: Status Derivation). Recalibration happens later, after real data is fetched — out of scope for this plan.
- No new signals beyond news mentions, founder mentions, app-store rating (spec: Out of Scope).
- No structural UI/animation changes beyond color retargeting (spec: Out of Scope).
- Brand palette (verbatim from spec, pulled from titancapital.vc's live CSS):
  - `--color-tc-navy: #172d4f` — primary
  - `--color-tc-coral: #e1624b` — accent
  - `--color-tc-blue: #1890d7` — secondary accent
  - `--color-tc-cream: #EEEADD` — background tint / glass base
- `FIRECRAWL_API_KEY` is read from `.env` (gitignored) and is never pasted into chat — the user adds it to `.env` themselves. No task in this plan touches `.env` contents or runs `npm run refresh-data` with a real key.
- New GitHub repo, new Vercel project, new Firecrawl account are the user's manual follow-up steps outside this repo — not part of this plan.

---

### Task 1: Verified Titan Capital company data

**Files:**
- Modify: `data/companies.ts`
- Test: `tests/companies.test.ts` (existing — no changes needed, assertions are generic)

**Interfaces:**
- Consumes: `Company` type from `lib/types.ts:1-7` (`{ slug, name, founders, newsQuery, appStore?: { iosId?, androidId? } }`) — unchanged.
- Produces: `COMPANIES: Company[]` — consumed unchanged by `scripts/refresh-data.ts`, `app/page.tsx`, `app/company/[slug]/page.tsx`, and `tests/companies.test.ts`.

App-store IDs below were verified by running each scraper's own `.search()` function against the live App Store / Play Store (not guessed) — see verification output referenced in this task. `iosId` is the numeric App Store ID (not the bundle identifier); `app-store-scraper`'s `.app({ id })` call expects the numeric form, matching the existing Swiggy/Paytm/etc. entries this replaces.

- [ ] **Step 1: Replace the full contents of `data/companies.ts`**

```typescript
import type { Company } from '../lib/types';

export const COMPANIES: Company[] = [
  {
    slug: 'razorpay',
    name: 'Razorpay',
    founders: ['Harshil Mathur', 'Shashank Kumar'],
    newsQuery: 'Razorpay',
    appStore: { iosId: '1497250144', androidId: 'com.razorpay.payments.app' },
  },
  {
    slug: 'ola',
    name: 'Ola',
    founders: ['Bhavish Aggarwal', 'Ankit Bhati'],
    newsQuery: 'Ola cabs',
    appStore: { iosId: '539179365', androidId: 'com.olacabs.customer' },
  },
  {
    slug: 'urban-company',
    name: 'Urban Company',
    founders: ['Abhiraj Bhal', 'Varun Khaitan', 'Raghav Chandra'],
    newsQuery: 'Urban Company',
    appStore: { iosId: '1032480595', androidId: 'com.urbanclap.urbanclap' },
  },
  {
    slug: 'mamaearth',
    name: 'Mamaearth',
    founders: ['Ghazal Alagh', 'Varun Alagh'],
    newsQuery: 'Mamaearth',
    appStore: { iosId: '1530385461', androidId: 'com.mamaearthapp' },
  },
  {
    slug: 'jupiter',
    name: 'Jupiter',
    founders: ['Jitendra Gupta'],
    newsQuery: 'Jupiter Money neobank',
    appStore: { iosId: '1507748747', androidId: 'money.jupiter' },
  },
  {
    slug: 'khatabook',
    name: 'Khatabook',
    founders: ['Ravish Naresh', 'Ashish Sonone', 'Dhanesh Kumar'],
    newsQuery: 'Khatabook',
    appStore: { iosId: '1488204139', androidId: 'com.vaibhavkalpe.android.khatabook' },
  },
  {
    slug: 'bewakoof',
    name: 'Bewakoof',
    founders: ['Prabhkiran Singh', 'Siddharth Munot'],
    newsQuery: 'Bewakoof fashion',
    appStore: { iosId: '1100190514', androidId: 'com.bewakoof.bewakoof' },
  },
  {
    slug: 'park-plus',
    name: 'Park+',
    founders: ['Amit Lakhotia'],
    newsQuery: 'Park+ parking app',
    appStore: { iosId: '1244749178', androidId: 'com.ovunque.parkwheels' },
  },
  {
    slug: 'teachmint',
    name: 'Teachmint',
    founders: ['Mihir Gupta', 'Payoj Jain', 'Divyansh Bordia', 'Namuduri Prasad'],
    newsQuery: 'Teachmint',
    appStore: { iosId: '1544210597', androidId: 'com.teachmint.teachmint' },
  },
  {
    slug: 'netmeds',
    name: 'Netmeds',
    founders: ['Pradeep Dadha'],
    newsQuery: 'Netmeds online pharmacy',
    appStore: { iosId: '1011070011', androidId: 'com.NetmedsMarketplace.Netmeds' },
  },
];
```

- [ ] **Step 2: Run the existing company data tests**

Run: `npx vitest run tests/companies.test.ts`
Expected: PASS — 3 tests (`has exactly 10 companies`, `has unique slugs`, `every company has at least one founder and a news query`).

- [ ] **Step 3: Commit**

```bash
git add data/companies.ts
git commit -m "feat: swap portfolio to Titan Capital's 10 tracked companies"
```

---

### Task 2: Retarget brand color tokens in globals.css

**Files:**
- Modify: `app/globals.css:11-17,22-24,30-32,38-67,76-91,97-107`

**Interfaces:**
- Consumes: nothing new.
- Produces: Tailwind `@theme inline` tokens `--color-tc-navy`, `--color-tc-navy-deep`, `--color-tc-blue`, `--color-tc-coral`, `--color-tc-glass-1`, `--color-tc-glass-2`, `--color-tc-glass-3`, `--color-tc-cream` — Task 3 renames every `ec-*` Tailwind utility class (e.g. `text-ec-blue`, `bg-ec-navy`) to the matching `tc-*` name, so these exact token names must match what Task 3 references.

The spec gives four brand colors (navy, coral, blue, cream) but the existing design system needs seven roles (navy, navy-deep, blue, three glass tints, cream) because the glassmorphism treatment layers multiple translucent blues. Map the spec's `tc-blue` (`#1890d7`) onto the existing `ec-blue` role (accents, links, glows), introduce `tc-coral` as a new token for status/CTA highlights per the spec, derive `tc-navy-deep` as a darker shade of `tc-navy` for the background gradient's second stop (same relationship `ec-navy-deep` had to `ec-navy`), and derive the three glass tints from `tc-blue` and `tc-cream` (same relationship the `ec-glass-*` tokens had to `ec-blue`/`ec-cream`) so the glass-card gradients keep the same visual proportions with the new hues.

- [ ] **Step 1: Replace the `:root` and `@theme inline` blocks**

```css
:root {
  --background: #172d4f;
  --foreground: #eeeadd;
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-tc-navy: #172d4f;
  --color-tc-navy-deep: #0f1e35;
  --color-tc-blue: #1890d7;
  --color-tc-coral: #e1624b;
  --color-tc-glass-1: #a8d4f0;
  --color-tc-glass-2: #cfe8f7;
  --color-tc-glass-3: #f3f0e6;
  --color-tc-cream: #eeeadd;
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
}

body {
  background-color: var(--color-tc-navy);
  color: var(--color-tc-cream);
  font-family: var(--font-sans), system-ui, sans-serif;
  -webkit-font-smoothing: antialiased;
}

::selection {
  background: rgba(24, 144, 215, 0.35);
  color: #f3f0e6;
}
```

- [ ] **Step 2: Replace the glassmorphism, sheen, grid, and gradient-text blocks**

```css
/* ---- Glassmorphism ---- */

.glass-card {
  position: relative;
  background: linear-gradient(
    135deg,
    rgba(168, 212, 240, 0.09) 0%,
    rgba(207, 232, 247, 0.04) 55%,
    rgba(24, 144, 215, 0.05) 100%
  );
  border: 1px solid rgba(168, 212, 240, 0.16);
  backdrop-filter: blur(18px) saturate(140%);
  -webkit-backdrop-filter: blur(18px) saturate(140%);
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.28),
    inset 0 1px 0 rgba(243, 240, 230, 0.10);
  transition:
    border-color 0.35s ease,
    box-shadow 0.35s ease,
    background 0.35s ease;
}

.glass-card:hover {
  border-color: rgba(24, 144, 215, 0.45);
  background: linear-gradient(
    135deg,
    rgba(168, 212, 240, 0.13) 0%,
    rgba(207, 232, 247, 0.06) 55%,
    rgba(24, 144, 215, 0.10) 100%
  );
  box-shadow:
    0 16px 48px rgba(0, 0, 0, 0.35),
    0 0 44px rgba(24, 144, 215, 0.18),
    inset 0 1px 0 rgba(243, 240, 230, 0.14);
}

/* Sheen that sweeps across a card on hover */
.glass-sheen::after {
  content: "";
  position: absolute;
  inset: 0;
  border-radius: inherit;
  background: linear-gradient(
    115deg,
    transparent 30%,
    rgba(243, 240, 230, 0.07) 48%,
    rgba(243, 240, 230, 0.12) 52%,
    transparent 70%
  );
  background-size: 260% 100%;
  background-position: 120% 0;
  opacity: 0;
  pointer-events: none;
  transition: background-position 0.9s cubic-bezier(0.22, 1, 0.36, 1), opacity 0.4s ease;
}

.glass-sheen:hover::after {
  background-position: -60% 0;
  opacity: 1;
}

/* Faint blueprint grid used by the parallax background */
.bg-grid-faint {
  background-image:
    linear-gradient(rgba(168, 212, 240, 0.045) 1px, transparent 1px),
    linear-gradient(90deg, rgba(168, 212, 240, 0.045) 1px, transparent 1px);
  background-size: 56px 56px;
  mask-image: radial-gradient(ellipse 90% 70% at 50% 20%, black 30%, transparent 78%);
  -webkit-mask-image: radial-gradient(ellipse 90% 70% at 50% 20%, black 30%, transparent 78%);
}

/* Gradient headline text */
.text-gradient-brand {
  background: linear-gradient(100deg, #eeeadd 20%, #a8d4f0 55%, #1890d7 100%);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}
```

Leave the `@keyframes status-pulse`, `.status-dot-pulse`, and `@media (prefers-reduced-motion: reduce)` blocks at the end of the file exactly as they are — they use `var(--pulse-color, ...)` set inline per-status in `StatusBadge.tsx`, not the `ec-*`/`tc-*` tokens, so nothing there changes.

- [ ] **Step 3: Start the dev server and confirm the page loads with no console errors**

Run: `npm run dev` (background), then load `http://localhost:3000` in a browser or via `curl -s http://localhost:3000 | head -5`
Expected: HTML response with no build errors in the terminal (Task 3 hasn't run yet, so `ec-*` class names in components will still render — that's expected at this point; Tailwind just won't have matching tokens for them yet, so those specific colors will fall back to unstyled, not error).

- [ ] **Step 4: Commit**

```bash
git add app/globals.css
git commit -m "feat: retarget color tokens to Titan Capital brand palette"
```

---

### Task 3: Rename `ec-*` Tailwind classes to `tc-*` across app and components

**Files:**
- Modify: `app/page.tsx:33-89`
- Modify: `app/layout.tsx:16-18`
- Modify: `components/CompanyCard.tsx:45-83`
- Modify: `components/CompanyDetailView.tsx:43-208`
- Modify: `components/ParallaxBackground.tsx:30-53`
- Modify: `components/RankSparkline.tsx:18-62`
- Modify: `components/StatusBadge.tsx` — no `ec-*` references exist here (colors come from `STATUS_COLORS` in `lib/status.ts`), confirm no change needed.

**Interfaces:**
- Consumes: `tc-navy`, `tc-navy-deep`, `tc-blue`, `tc-coral`, `tc-glass-1`, `tc-glass-2`, `tc-glass-3`, `tc-cream` Tailwind tokens produced by Task 2.
- Produces: no new exports — this task only renames existing Tailwind utility class strings and copy text; component signatures are unchanged.

Every occurrence of an `ec-` prefixed Tailwind class becomes the same class with `tc-` in its place (e.g. `text-ec-blue` → `text-tc-blue`, `bg-ec-glass-1/8` → `bg-tc-glass-1/8`, `outline-ec-blue` → `outline-tc-blue`). No `ec-coral` existed before — Task 2's new `tc-coral` token isn't referenced by any component yet, since the spec calls for color-token changes only, not new visual elements; it's available for future use but no task in this plan wires it in.

- [ ] **Step 1: Rename `app/page.tsx`**

Replace lines 33-89 of `app/page.tsx`:

```tsx
        <header>
          <p className="text-xs font-semibold uppercase tracking-[0.28em] text-tc-blue">
            Titan Capital
          </p>
          <h1 className="text-gradient-brand mt-3 max-w-2xl text-4xl font-bold tracking-tight sm:text-5xl">
            Portfolio Pulse
          </h1>
          <p className="mt-4 max-w-xl text-base leading-relaxed text-tc-glass-1/70">
            Public momentum signals across the portfolio — news coverage,
            founder mentions, and app-store standing at a glance.
          </p>

          <div className="mt-7 flex flex-wrap items-center gap-x-5 gap-y-2 text-sm text-tc-glass-1/60">
            <span>
              Last refreshed{' '}
              <time
                dateTime={snapshot.generatedAt}
                className="font-medium text-tc-glass-2"
              >
                {TIMESTAMP_FORMAT.format(new Date(snapshot.generatedAt))} IST
              </time>
            </span>
            <span className="hidden h-1 w-1 rounded-full bg-tc-glass-1/30 sm:inline-block" />
            <span className="flex items-center gap-4">
              {STATUS_ORDER.map((status) => (
                <span key={status} className="flex items-center gap-1.5">
                  <span
                    className="font-semibold tabular-nums"
                    style={{ color: STATUS_COLORS[status] }}
                  >
                    {statusCounts[status]}
                  </span>
                  <span className="capitalize">{status}</span>
                </span>
              ))}
            </span>
          </div>
        </header>

        <div className="mt-12 grid grid-cols-1 gap-5 sm:grid-cols-2 sm:gap-6 lg:grid-cols-3">
          {COMPANIES.map((company, index) => {
            const signals = snapshot.companies.find(
              (c) => c.slug === company.slug,
            );
            if (!signals) return null;
            return (
              <CompanyCard
                key={company.slug}
                name={company.name}
                slug={company.slug}
                signals={signals}
                index={index}
              />
            );
          })}
        </div>

        <footer className="mt-16 border-t border-tc-glass-1/10 pt-6 text-xs text-tc-glass-1/40">
          Signals are derived from public sources and refreshed on a schedule.
          Static snapshot — not investment advice.
        </footer>
      </div>
    </main>
  );
}
```

- [ ] **Step 2: Update `app/layout.tsx` metadata**

Replace lines 15-19 of `app/layout.tsx`:

```tsx
export const metadata: Metadata = {
  title: "Titan Capital Portfolio Pulse",
  description:
    "Public momentum signals across Titan Capital's portfolio companies.",
};
```

- [ ] **Step 3: Rename `components/CompanyCard.tsx`**

Replace lines 45-92:

```tsx
        className="group block h-full rounded-2xl focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-tc-blue"
        aria-label={`${name} — signal details`}
      >
        <div className="glass-card glass-sheen flex h-full flex-col rounded-2xl p-6">
          <div className="flex items-start justify-between gap-3">
            <div className="flex items-center gap-3.5">
              {/* Monogram roundel */}
              <div className="flex h-11 w-11 shrink-0 items-center justify-center rounded-xl border border-tc-glass-1/20 bg-gradient-to-br from-tc-blue/25 to-tc-blue/5 text-base font-semibold text-tc-glass-2 shadow-[inset_0_1px_0_rgba(243,240,230,0.15)]">
                {name.charAt(0)}
              </div>
              <h3 className="text-lg font-semibold leading-snug tracking-tight text-tc-cream">
                {name}
              </h3>
            </div>
            <StatusBadge status={status} />
          </div>

          <dl className="mt-5 space-y-2 border-t border-tc-glass-1/10 pt-4 text-sm">
            <div className="flex items-baseline justify-between">
              <dt className="text-tc-glass-1/60">News mentions</dt>
              <dd className="font-medium tabular-nums text-tc-glass-2">
                {signals.news === null ? '—' : newsCount}
              </dd>
            </div>
            <div className="flex items-baseline justify-between">
              <dt className="text-tc-glass-1/60">Founder mentions</dt>
              <dd className="font-medium tabular-nums text-tc-glass-2">
                {signals.founderMentions === null ? '—' : founderCount}
              </dd>
            </div>
            <div className="flex items-baseline justify-between">
              <dt className="text-tc-glass-1/60">App rating</dt>
              <dd className="font-medium tabular-nums text-tc-glass-2">
                {rating !== null ? `★ ${rating.toFixed(1)}` : '—'}
              </dd>
            </div>
          </dl>

          <div className="mt-auto flex items-center gap-1 pt-5 text-sm font-medium text-tc-blue/80 transition-colors group-hover:text-tc-blue">
            View signals
            <span
              aria-hidden
              className="inline-block transition-transform duration-300 ease-out group-hover:translate-x-1"
            >
              &rarr;
            </span>
          </div>
        </div>
      </Link>
    </motion.div>
  );
}
```

- [ ] **Step 4: Rename `components/CompanyDetailView.tsx`**

Replace lines 42-91 (the `HeadlineList` function body):

```tsx
  if (headlines === null) {
    return (
      <p className="text-sm text-tc-glass-1/60">
        &mdash; Not fetched yet. Headlines will appear after the next data
        refresh.
      </p>
    );
  }

  if (headlines.length === 0) {
    return <p className="text-sm text-tc-glass-1/60">{emptyLabel}</p>;
  }

  return (
    <ul className="space-y-3">
      {headlines.map((h) => (
        <li key={h.url}>
          <a
            href={h.url}
            target="_blank"
            rel="noopener noreferrer"
            className="glass-card glass-sheen group flex items-baseline justify-between gap-4 rounded-2xl px-5 py-4 focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-tc-blue"
          >
            <span>
              <span className="block text-sm font-medium leading-snug text-tc-cream transition-colors group-hover:text-tc-glass-2">
                {h.title}
              </span>
              {h.publishedAt && (() => {
                const { display, iso } = formatPublishedAt(h.publishedAt);
                return (
                  <time
                    dateTime={iso ?? undefined}
                    className="mt-1.5 block text-xs text-tc-glass-1/50"
                  >
                    {display}
                  </time>
                );
              })()}
            </span>
            <span
              aria-hidden
              className="shrink-0 text-sm text-tc-blue/70 transition-all duration-300 ease-out group-hover:translate-x-0.5 group-hover:-translate-y-0.5 group-hover:text-tc-blue"
            >
              &#8599;
            </span>
          </a>
        </li>
      ))}
    </ul>
  );
}
```

Replace lines 139-208 (from the "Portfolio Pulse" back-link through the closing `</div>`):

```tsx
        <Link
          href="/"
          className="group inline-flex items-center gap-1.5 rounded-lg text-sm font-medium text-tc-glass-1/60 transition-colors hover:text-tc-blue focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-tc-blue"
        >
          <span
            aria-hidden
            className="inline-block transition-transform duration-300 ease-out group-hover:-translate-x-1"
          >
            &larr;
          </span>
          Portfolio Pulse
        </Link>
      </motion.div>

      <motion.header className="mt-8" {...enter(0.08)}>
        <div className="flex items-start justify-between gap-4">
          <div className="flex items-center gap-4">
            {/* Monogram roundel — same language as the dashboard cards */}
            <div className="flex h-14 w-14 shrink-0 items-center justify-center rounded-2xl border border-tc-glass-1/20 bg-gradient-to-br from-tc-blue/25 to-tc-blue/5 text-2xl font-semibold text-tc-glass-2 shadow-[inset_0_1px_0_rgba(243,240,230,0.15)]">
              {company.name.charAt(0)}
            </div>
            <h1 className="text-gradient-brand text-4xl font-bold tracking-tight sm:text-5xl">
              {company.name}
            </h1>
          </div>
          <div className="pt-2">
            <StatusBadge status={status} />
          </div>
        </div>

        <dl className="mt-6 flex flex-wrap items-baseline gap-x-8 gap-y-2 text-sm">
          <div className="flex items-baseline gap-2">
            <dt className="text-xs font-semibold uppercase tracking-[0.18em] text-tc-glass-1/50">
              {company.founders.length > 1 ? 'Founders' : 'Founder'}
            </dt>
            <dd className="font-medium text-tc-glass-2">
              {company.founders.join(', ')}
            </dd>
          </div>
          <div className="flex items-baseline gap-2 text-tc-glass-1/50">
            <dt>Last refreshed</dt>
            <dd>
              <time dateTime={generatedAt} className="text-tc-glass-1/70">
                {TIMESTAMP_FORMAT.format(new Date(generatedAt))} IST
              </time>
            </dd>
          </div>
        </dl>
      </motion.header>

      {sections.map((section, i) => (
        <motion.section
          key={section.title}
          className="mt-12"
          {...enter(0.16 + i * 0.08)}
        >
          <h2 className="text-xs font-semibold uppercase tracking-[0.28em] text-tc-blue">
            {section.title}
          </h2>
          <div className="mt-4">{section.body}</div>
        </motion.section>
      ))}

      <motion.footer
        className="mt-16 border-t border-tc-glass-1/10 pt-6 text-xs text-tc-glass-1/40"
        {...enter(0.16 + sections.length * 0.08)}
      >
        Signals are derived from public sources and refreshed on a schedule.
        Static snapshot &mdash; not investment advice.
      </motion.footer>
    </div>
  );
}
```

- [ ] **Step 5: Rename `components/ParallaxBackground.tsx`**

Replace lines 27-64:

```tsx
    <div
      aria-hidden
      className="pointer-events-none fixed inset-0 -z-10 overflow-hidden bg-gradient-to-b from-tc-navy via-tc-navy-deep to-black"
    >
      {/* Far layer: aurora wash at the top + blueprint grid */}
      <motion.div style={{ y: yFar }} className="absolute inset-0">
        <div
          className="absolute -top-40 left-1/2 h-[34rem] w-[72rem] -translate-x-1/2 rounded-[100%] opacity-60 blur-3xl"
          style={{
            background:
              'radial-gradient(ellipse at center, rgba(24,144,215,0.16) 0%, rgba(24,144,215,0.05) 45%, transparent 70%)',
          }}
        />
        <div className="bg-grid-faint absolute inset-0" />
      </motion.div>

      {/* Mid layer: large drifting orbs */}
      <motion.div style={{ y: yMid }} className="absolute inset-0">
        <div className="absolute left-[12%] top-[28%] h-[26rem] w-[26rem] rounded-full bg-tc-blue/12 blur-3xl" />
        <div className="absolute right-[8%] top-[62%] h-[20rem] w-[20rem] rounded-full bg-tc-glass-2/10 blur-3xl" />
      </motion.div>

      {/* Near layer: smaller, brighter accents moving against the scroll */}
      <motion.div style={{ y: yNear }} className="absolute inset-0">
        <div className="absolute right-[24%] top-[18%] h-40 w-40 rounded-full bg-tc-blue/15 blur-2xl" />
        <div className="absolute bottom-[8%] left-[30%] h-56 w-56 rounded-full bg-tc-glass-1/8 blur-3xl" />
      </motion.div>

      {/* Vignette to keep edges dark and cards legible */}
      <div
        className="absolute inset-0"
        style={{
          background:
            'radial-gradient(ellipse 120% 90% at 50% 40%, transparent 55%, rgba(5,10,18,0.55) 100%)',
        }}
      />
    </div>
  );
}
```

- [ ] **Step 6: Rename `components/RankSparkline.tsx`**

Replace lines 16-71 (the whole function body):

```tsx
  if (ranks === null) {
    return (
      <p className="text-sm text-tc-glass-1/60">
        &mdash; App-store data hasn&rsquo;t been fetched yet. It will appear
        after the next data refresh.
      </p>
    );
  }

  if (ranks.length === 0) {
    return (
      <p className="text-sm text-tc-glass-1/60">
        No app-store listing tracked for this company.
      </p>
    );
  }

  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-2">
      {ranks.map((r) => (
        <div
          key={r.platform}
          className="glass-card glass-sheen rounded-2xl px-5 py-4"
        >
          <p className="text-[11px] font-semibold uppercase tracking-[0.18em] text-tc-glass-1/60">
            {PLATFORM_LABEL[r.platform]}
          </p>
          <div className="mt-2 flex items-baseline gap-3">
            <p className="text-3xl font-semibold tabular-nums tracking-tight text-tc-cream">
              {r.rating !== null ? (
                <>
                  <span aria-hidden className="mr-1 text-xl text-tc-blue">
                    &#9733;
                  </span>
                  {r.rating.toFixed(1)}
                </>
              ) : (
                <span className="text-tc-glass-1/50">&mdash;</span>
              )}
            </p>
            {r.rank !== null && (
              <p className="text-sm text-tc-glass-1/60">
                #{r.rank} in category
              </p>
            )}
          </div>
          <p className="mt-2 text-xs text-tc-glass-1/40">
            {r.rating !== null || r.rank !== null
              ? 'Store rating'
              : 'No rating reported'}
          </p>
        </div>
      ))}
    </div>
  );
}
```

- [ ] **Step 7: Grep to confirm no `ec-` classes remain**

Run: `grep -rn "ec-" app/ components/ | grep -v node_modules`
Expected: no output (empty).

- [ ] **Step 8: Run the full test suite**

Run: `npx vitest run`
Expected: PASS — all existing tests in `tests/companies.test.ts`, `tests/status.test.ts`, `tests/fetch-news.test.ts`, `tests/fetch-app-rank.test.ts` (none of these test UI, so this run mainly guards against a typo breaking an import elsewhere).

- [ ] **Step 9: Build the static export to confirm no TypeScript/JSX errors**

Run: `npm run build`
Expected: build completes successfully, producing output in `out/`.

- [ ] **Step 10: Start the dev server and visually confirm the rebrand**

Run: `npm run dev` (background), then load `http://localhost:3000`
Expected: page shows "Titan Capital" / "Portfolio Pulse" header, navy/coral/blue color scheme, 10 company cards (Razorpay, Ola, Urban Company, Mamaearth, Jupiter, Khatabook, Bewakoof, Park+, Teachmint, Netmeds) with "—" placeholders for news/founder/rating (no `data/portfolio.json` exists yet, so `app/page.tsx`'s import of it will fail at build time — see Task 4 for the placeholder snapshot needed to unblock this).

- [ ] **Step 11: Commit**

```bash
git add app/page.tsx app/layout.tsx components/CompanyCard.tsx components/CompanyDetailView.tsx components/ParallaxBackground.tsx components/RankSparkline.tsx
git commit -m "feat: rename brand color tokens and copy from Elevation to Titan Capital"
```

---

### Task 4: Placeholder data snapshot, project metadata, and docs

**Files:**
- Create: `data/portfolio.json`
- Modify: `package.json:2`
- Modify: `README.md` (full rewrite)

**Interfaces:**
- Consumes: `PortfolioSnapshot` type from `lib/types.ts:29-32` (`{ generatedAt: string, companies: CompanySignals[] }`) and `CompanySignals` (`{ slug, news, founderMentions, appRanks }`) from `lib/types.ts:22-27`.
- Produces: `data/portfolio.json` — read by `app/page.tsx:2,8` and `app/company/[slug]/page.tsx:4,9` via `import portfolio from '../data/portfolio.json'`; both files build without a real Firecrawl-fetched snapshot present, matching how the existing Elevation build works before its first `refresh-data` run.

`data/portfolio.json` is a build-time dependency of both page components (static import, not a runtime fetch) — without it, `npm run build` and `npm run dev` fail immediately with a module-not-found error, before either page renders. This task writes a placeholder snapshot with all-null signals for the 10 Titan companies (mirroring `CompanySignals` with `news: null, founderMentions: null, appRanks: null`, which the UI already renders as "—" per `lib/types.ts` and the null-vs-zero handling in `CompanyCard.tsx:66,72,77` and `RankSparkline.tsx:16-22`), so the site builds and displays correctly before the user ever runs `npm run refresh-data` with their own Firecrawl key.

- [ ] **Step 1: Write `data/portfolio.json`**

```json
{
  "generatedAt": "2026-07-03T00:00:00.000Z",
  "companies": [
    { "slug": "razorpay", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "ola", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "urban-company", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "mamaearth", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "jupiter", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "khatabook", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "bewakoof", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "park-plus", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "teachmint", "news": null, "founderMentions": null, "appRanks": null },
    { "slug": "netmeds", "news": null, "founderMentions": null, "appRanks": null }
  ]
}
```

- [ ] **Step 2: Update `package.json` name**

Replace line 2 of `package.json`:

```json
  "name": "titan-capital-portfolio-pulse",
```

- [ ] **Step 3: Rewrite `README.md`**

```markdown
# Titan Capital Portfolio Pulse

A static dashboard tracking public momentum signals — news coverage,
founder mentions, and app-store rank — across 10 Titan Capital
portfolio companies. Built as a portfolio piece for a VC-adjacent
interview.

## Running locally

\`\`\`bash
npm install
npm run dev
\`\`\`

Open [http://localhost:3000](http://localhost:3000).

## Refreshing data

\`\`\`bash
npm run refresh-data
\`\`\`

This re-scrapes news, founder mentions, and app-store ranks into
`data/portfolio.json`. It requires a `FIRECRAWL_API_KEY` environment
variable — get a free Firecrawl account (no credit card needed for the
free tier used here) and copy `.env.example` to `.env` with your key.
Without the key, the script skips news/founder fetching and only
refreshes app-store ranks.

## Build & deploy

\`\`\`bash
npm run build
\`\`\`

This produces a static export in `out/`, deployable to Vercel's free
Hobby tier (or any static host). This repo's root *is* the Next.js
app root, so no Root Directory override is needed at Vercel import time.
```

- [ ] **Step 4: Run the full test suite and build one more time to confirm everything is wired together**

Run: `npx vitest run && npm run build`
Expected: all tests pass, build completes with no errors, `out/index.html` and `out/company/*/index.html` exist for all 10 slugs.

- [ ] **Step 5: Commit**

```bash
git add data/portfolio.json package.json README.md
git commit -m "chore: add placeholder snapshot and update project metadata for Titan Capital"
```

---

## After this plan

Once merged, the remaining steps are manual and outside this repo's code:

1. Create a new Firecrawl account/API key for Titan Capital (separate from Elevation's) and add it to a local `.env` (never commit it).
2. Run `npm run refresh-data` locally to fetch real signals, then check the news-count distribution — recalibrate `lib/status.ts`'s thresholds if it skews all-one-bucket, same issue hit on the Elevation build.
3. Push to a new GitHub repo (separate from Elevation's) and import into a new Vercel project on the Hobby tier.
