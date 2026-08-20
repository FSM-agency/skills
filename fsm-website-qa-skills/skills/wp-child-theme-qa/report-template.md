# Report template — WordPress / Divi Child Theme QA

Use this structure for the final user-facing report. Fill every section; use "None found" when clean. Use "Skipped — …" when a check could not run.

```markdown
# [SITE_NAME] Child Theme / Code Hygiene QA Report
**Target URL:** [SITE_URL]
**SSH:** [SSH_TARGET / already connected]
**Child theme:** [CHILD_THEME_PATH]
**Date:** [YYYY-MM-DD]
**Mode:** Read-only (no writes, no data-altering WP-CLI)
**Artifacts (local only):** `[OUTPUT_DIR]` (project-local; never on Kinsta)

---

## Step 0 — Environment Recon

| Check | Result |
|-------|--------|
| Shell user / access | … |
| WP-CLI | Available / Missing |
| Child theme path | … |
| Parent theme | … |
| WP_DEBUG / WP_DEBUG_LOG | … |
| debug.log | path, size, or absent |

**Degraded steps (if any):** …

---

## 1. Theme Structure

| Severity | Track | Finding | Location | Evidence |
|----------|-------|---------|----------|----------|
| … | Developer Action / Coaching Note | … | … | … |

**Pass notes:** child vs parent, where CSS/PHP live, custom code outside child theme.

---

## 2. CSS Practices

| Severity | Track | Finding | Location | Evidence |
|----------|-------|---------|----------|----------|
| … | … | Centralization / tokens / !important / media / numbered Divi / `.fsm_*` / Swiper states | … | counts / samples |

**Token summary:** literals vs variables for each brand token (or skipped — none provided).

**Swiper:** Not present / present — custom arrow states checked (default, hover, focus, active, disabled).

---

## 3. Code Hygiene

| Severity | Track | Finding | Location | Evidence |
|----------|-------|---------|----------|----------|
| … | … | placeholder / debug / secret | path:line | excerpt |

---

## 4. PHP, Security & Agency Standards

| Severity | Track | Finding | Location | Evidence |
|----------|-------|---------|----------|----------|
| … | … | unsanitized input / unescaped echo / unprepared SQL / AJAX nonce / enqueue / FacetWP split / duplicate jQuery or Swiper | … | … |

---

## 5. Plugin Audit

| Plugin | Status | Version | Update available | Notes |
|--------|--------|---------|------------------|-------|
| … | active/inactive | … | yes/no | overlap / outdated / OK |

| Severity | Track | Finding | Evidence |
|----------|-------|---------|----------|
| … | … | inactive / redundant / outdated | … |

---

## 6. Error Log Review

**Status:** Reviewed / Not reviewed (logging off or file absent) / Skipped

| Severity | Track | Finding | Attribution | Evidence |
|----------|-------|---------|-------------|----------|
| … | … | unique message; PHP 8.1+ child/custom-plugin → Critical | child / plugin / core | count + sample |

---

## 7. Maintainability & Naming

| Severity | Track | Finding | Location | Evidence |
|----------|-------|---------|----------|----------|
| … | … | `.fsm_*` CSS (fail) / PHP prefix optional / shortcode docs / editability / workaround | … | … |

---

## 8. Figma Design Review

**Status:** Not run — no Figma URL provided  
*(or)* Completed / Skipped — Figma MCP unavailable / unauthenticated

| Severity | Track | Finding | Figma vs code | Evidence |
|----------|-------|---------|---------------|----------|
| … | … | token or layout drift | … | variable name / screenshot note |

---

## Known / Expected Content Gaps (Still Present — Not New Bugs)

| Item | Status |
|------|--------|
| … | Still present / Fixed / Not observed |

---

## Priority Fix List (Developer Action Only)

Ordered Critical → Minor. New issues only (exclude Known/Expected).

1. **Critical** — …
2. **Moderate** — …
3. **Minor** — …

---

## Coaching Notes

Process/habit items for the build team (not launch blockers unless escalated above).

1. …
2. …

---

## Skipped Checks

| Check | Reason |
|-------|--------|
| … | tooling missing / logging off / no FIGMA_URL / … |
```
