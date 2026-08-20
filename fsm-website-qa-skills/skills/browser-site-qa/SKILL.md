---
name: browser-site-qa
description: >-
  Runs an automated browser/frontend QA pass against a live URL using Playwright,
  axe-core, and Lighthouse. Covers link crawl, console errors, accessibility,
  performance, SEO/meta, forms, Swiper states, and responsive screenshots. Use
  when the user asks for browser QA, frontend QA, site QA, Lighthouse audit,
  accessibility audit, or a visual/responsive QA pass on a staging or production
  URL. Run locally or on a Cursor cloud sandbox — never on Kinsta (no npm).
  Does not cover SSH/codebase hygiene — that is wp-child-theme-qa.
---

# Browser / Frontend Site QA

Automated, evidence-backed QA against a **live URL**. Browser half only — do **not** duplicate SSH/codebase hygiene.

## Runtime (non-negotiable)

**Do not run this skill on Kinsta.** Staging/production hosts do not have npm. Never `npm install` / `npx playwright` / `lighthouse` on the server.

Allowed: developer **local** machine (Cursor desktop) or a **Cursor cloud sandbox** with Node + npm. Hit `TARGET_URL` over HTTPS.

If the current shell is SSH’d into Kinsta (`pwd` like `/www/.../public`): **stop** this pass. Use `wp-child-theme-qa` for SSH; run this skill from local or cloud.

## Repeatability

Same inputs → same check set. Documented caps. Skip with a reason — never silent omit. Never invent `KNOWN_EXPECTED_ISSUES`. Never use a prior client as defaults.

## Inputs — ask if missing

Parse the user message first. **Do not start** until required fields are known. Do not invent URLs or site names.

1. Use any field already provided.
2. **Required and missing** — ask in one message (only missing required fields).
3. **Optional and missing** — ask once in the same message, each with skip/default below.
4. After answers or skips, proceed. Do not re-ask fields already given.

| Input | Required | Skip / default |
|-------|----------|----------------|
| `TARGET_URL` | Yes | — |
| `SITE_NAME` / client | Yes | Derive from URL only if the user agrees when asked |
| `PLATFORM` | No | Detect (WordPress, Shopify, custom) |
| `KNOWN_EXPECTED_ISSUES` | No | Empty list — never invent |
| `KEY_PAGES` | No | Discover via sitemap / nav / REST |
| `OUTPUT_DIR` | No | `<project-root>/reports/<slug>-qa/` |

`<project-root>` = Cursor workspace / git root **on the machine running Node**. Never `~/reports`. Honor an explicit `OUTPUT_DIR`.

**Known expected issues:** Note if still present. Do **not** flag them as new bugs. Put them in "Known / Expected".

## Step 0 — Tooling (local or Cursor cloud only)

Confirm on the **agent machine** (install if missing — never on Kinsta):

- Node.js + npm
- Playwright + Chromium
- Lighthouse CLI
- axe-core via `@axe-core/playwright`

Create `OUTPUT_DIR` with `output/`, `output/screenshots/`, `output/lighthouse/`. Prefer a throwaway Node project **under `OUTPUT_DIR`**. Do not commit secrets or that throwaway project unless the user asks.

Recommend the **client repo** `.gitignore` include `reports/`.

## Workflow checklist

```
QA Progress:
- [ ] 0 Tooling ready (local or cloud — not Kinsta)
- [ ] 1 Link crawl
- [ ] 2 Console errors
- [ ] 3 Accessibility (axe)
- [ ] 4 Performance (Lighthouse) + safe recs
- [ ] 5 SEO / meta
- [ ] 6 Forms
- [ ] 7 Responsive screenshots
- [ ] 8 Swiper (if present)
- [ ] 9 WordPress extras
- [ ] Report delivered
```

### 1. Link crawl

- Discover pages: sitemap.xml, nav, platform APIs (e.g. WordPress `/wp-json/wp/v2/pages`).
- Crawl a capped set (~40–80; prioritize `KEY_PAGES`).
- Flag **4xx/5xx** as Critical (Moderate if clearly orphaned drafts).
- Spot-check in-content CTAs, not only primary nav.
- Note external links; do not deep-crawl unless they look broken from the referrer.

**Do not visit / do not mutate:** `wp-admin`, `wp-login.php`, logout/nonce URLs, `?preview=true` / customizer preview, `add-to-cart`, checkout/donate POSTs, non-HTML feeds. Record as skipped — not 4xx failures.

**Consent overlay:** If a cookie banner blocks the viewport, document it. Do not treat Lighthouse/axe failures caused **only** by the overlay as launch bugs. Re-run key pages after dismiss only if dismiss is a simple click with no legal side effects. If unclear, skip re-run and note the limitation.

### 2. Console errors

Playwright: `console` type `error` and `pageerror` on key pages. Ignore documented benign third-party noise. Flag first-party / blocking errors.

### 3. Accessibility

axe-core tags `wcag2a`, `wcag2aa`, `wcag21a`, `wcag21aa` on key pages.

Map impact: `critical`/`serious` → Critical or Moderate; `moderate` → Moderate; `minor` → Minor. Include rule id, help, node count, sample selectors.

### 4. Performance (Lighthouse)

Mobile **and** desktop on homepage (optionally 1–2 heavy pages). Categories: performance, accessibility, best-practices, seo.

Report scores + FCP, LCP, TBT, CLS, Speed Index, TTI. Call out high-impact failed audits.

**Recommendations (advisory — do not implement on the server):** Suggest **safe** Early Hints / `preconnect` / `preload` / `prefetch` only when they will not break required behavior.

Do **not** recommend:

- prefetch/preload that races or duplicates Gravity Forms, reCAPTCHA, Stripe, or other form/payment scripts
- `modulepreload` / aggressive prefetch of `gravityforms.js` / `jquery` that can change load order
- prefetching next navigations that include checkout/donate/form POSTs

Prefer: `preconnect` to CDNs/fonts **already used**; `preload` for the actual homepage LCP image; Early Hints only for those same stable assets. Coaching Note unless missing LCP preload is clearly the homepage bottleneck (then Moderate).

### 5. SEO / meta

Per key page: title, meta description, H1 count/text, canonical, robots, og:title/description/image, missing image alts count, `lang`.

**Staging / `*.kinsta.cloud` / obvious dev hosts:** `noindex` is **expected** — note it, not a launch bug.

### 6. Forms

Locate Gravity Forms, native, Shopify, embeds. Field counts, submit present, placeholders. Empty/invalid submit for client-side validation when safe.

**Do not** complete real payments, donations, or spam live inboxes unless the user confirms a safe test path.

Ticket/purchase pages that never reach `networkidle`: `domcontentloaded` + timed wait.

### 7. Responsive screenshots

Viewports: ~1440×900, ~768×1024, ~375×812. Homepage + 3–4 priority pages (above-the-fold unless a layout bug needs full page). Note mobile nav, hamburger, clipped content, empty templates.

### 8. Swiper (if present)

If Swiper markup/classes exist (`swiper`, `swiper-button-next/prev`, `swiper-pagination`):

- Exercise next/prev at **start, middle, and end** of the track.
- Screenshot disabled vs hover; keyboard-focus the controls.
- If arrows are custom-styled, flag states that look identical: default, `:hover`, `:focus` / `:focus-visible`, `:active`, `[disabled]` / `.swiper-button-disabled`, locked/end if used.
- Missing states → Moderate; invisible/unusable arrows → Critical.
- If Swiper is absent, write `Not run — no Swiper`.

### 9. WordPress extras (when PLATFORM is WordPress / detected)

- Mixed content: `http://` assets on HTTPS.
- Max Mega Menu / primary nav: mobile toggle (`.mega-menu-toggle`) open/close.
- FacetWP: if present, listing still renders without depending on filter DOM; filters and listing are separate UI.
- Gravity Forms / Divi contact: submit present, required fields, no live spam (see step 6).
- `target=_blank` without `rel="noopener"` or `rel="noreferrer"` → Minor unless payment/PII flow.

## Severity rubric

| Severity | Use when |
|----------|----------|
| **Critical** | Broken primary journeys (404 on main CTA, missing contact form, severe a11y on core filters, unusable Swiper, catastrophic homepage LCP) |
| **Moderate** | Incomplete SEO on key pages, meaningful a11y, mobile layout, render-blocking, missing Swiper states, missing LCP preload bottleneck |
| **Minor** | Thin meta, non-descriptive links, `rel=noopener`, polish, low-impact a11y |

## Explicit exclusions

- No SSH, WP-CLI, DB, or child-theme/file edits.
- No npm/Playwright/Lighthouse **on Kinsta**.
- No inventing `KNOWN_EXPECTED_ISSUES`.
- Do not flag content the user marked as expected.

## Output

1. Write artifacts under `OUTPUT_DIR` (JSON, Lighthouse JSON, axe, screenshots).
2. Deliver one markdown report using [report-template.md](report-template.md).
3. End with **Priority Fix List (new issues only)** Critical → Minor.
4. Cite artifact paths for tickets.

## Platform hints

**WordPress:** REST + sitemap; Gravity Forms / Divi contact; Mega Menu `.mega-menu-toggle`; FacetWP listing vs filters.

**Shopify (secondary):** sitemap + nav; theme sections, cart drawer, app-embed console noise.

**Kinsta staging:** `x-robots-tag: noindex` expected; cache headers are not a QA fail by themselves.
