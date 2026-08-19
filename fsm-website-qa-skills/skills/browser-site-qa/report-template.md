# Report template — Browser / Frontend QA

Use this structure for the final user-facing report. Fill every section; use "None found" when clean. Use "Skipped — …" when a check could not run.

```markdown
# [SITE_NAME] Browser QA Report
**Target:** [TARGET_URL]
**Date:** [YYYY-MM-DD]
**Runtime:** Local / Cursor cloud (not Kinsta)
**Tooling:** Node · Playwright · Lighthouse · axe-core
**Artifacts:** `[OUTPUT_DIR]` (project-local)

---

## Step 0 — Environment

| Tool | Status |
|------|--------|
| Runtime | Local / Cursor cloud — not Kinsta |
| Node / npm | … |
| Playwright + Chromium | … |
| Lighthouse CLI | … |

**Consent overlay:** none / present (documented; overlay-only LH/axe fails not launch bugs)

---

## 1. Link Crawl

| Severity | Issue | Page(s) |
|----------|-------|---------|
| … | … | … |

**Pass notes:** pages crawled, sitemap, skipped URLs (`wp-admin`, login, preview, add-to-cart, etc.).

---

## 2. Console Errors

| Severity | Issue | Page(s) | Excerpt |
|----------|-------|---------|---------|
| … | … | … | `…` |

---

## 3. Accessibility

| Severity | Issue | Page(s) | Reference |
|----------|-------|---------|-----------|
| … | rule id + help | … | selectors / node count |

**Lighthouse a11y (if run):** Desktop … · Mobile …

---

## 4. Performance (Lighthouse)

### Scores

| Category | Desktop | Mobile |
|----------|---------|--------|
| Performance | … | … |
| Accessibility | … | … |
| Best Practices | … | … |
| SEO | … | … |

### Core metrics (homepage)

| Metric | Desktop | Mobile |
|--------|---------|--------|
| FCP | … | … |
| LCP | … | … |
| TBT | … | … |
| CLS | … | … |
| Speed Index | … | … |

| Severity | Issue | Page(s) | Reference |
|----------|-------|---------|-----------|
| … | … | … | audit id / savings |

### Performance recommendations (safe only)

Advisory only — not implemented in this pass. No GF/reCAPTCHA/Stripe/jquery load-order changes.

| Severity | Recommendation | Why it is safe |
|----------|----------------|----------------|
| Coaching Note / Moderate | preconnect / preload LCP / Early Hints / skip | … |

---

## 5. SEO / Meta

| Severity | Issue | Page(s) |
|----------|-------|---------|
| … | … | … |

Note staging `noindex` here as expected when applicable.

---

## 6. Forms

| Severity | Form | Status | Page |
|----------|------|--------|------|
| … | … | present / missing / validation notes | … |

---

## 7. Responsive Screenshots

| Severity | Issue | Viewport | Reference |
|----------|-------|----------|-----------|
| … | … | mobile/tablet/desktop | `screenshots/…` |

---

## 8. Swiper

**Status:** Not run — no Swiper / Completed

| Severity | Issue | State / position | Reference |
|----------|-------|------------------|-----------|
| … | missing hover/focus/disabled, etc. | start / middle / end | `screenshots/…` |

---

## 9. WordPress extras

| Severity | Issue | Page(s) | Reference |
|----------|-------|---------|-----------|
| … | mixed content / mega menu / FacetWP / target=_blank rel | … | … |

---

## Known / Expected Issues (Still Present — Not New Bugs)

| Item | Status |
|------|--------|
| … | Still present / Fixed / Not observed |

---

## Priority Fix List (New Issues Only)

1. **Critical** — …
2. **Moderate** — …
3. **Minor** — …
```
