---
name: wp-child-theme-qa
description: >-
  Runs a read-only WordPress/Divi child-theme code hygiene QA pass over SSH.
  Covers theme structure, CSS practices, PHP/security, FacetWP shortcode split,
  Swiper states, placeholder/debug/secret greps, plugin audit, PHP 8.1+ debug.log,
  maintainability, and optional Figma token drift. Use when the user asks for
  child theme QA, code hygiene pass, WordPress/Divi codebase audit, SSH QA,
  plugin audit, or debug.log review. Does not cover browser/Lighthouse — use
  browser-site-qa (local or Cursor cloud, never on Kinsta).
---

# WordPress / Divi Child Theme QA (SSH)

Read-only codebase hygiene against a **development site over SSH**. Code half only — do **not** duplicate Playwright / Lighthouse / axe (`browser-site-qa`).

## Repeatability

Same inputs → same check set. Documented caps. Skip with a reason — never silent omit. Never invent `KNOWN_CONTENT_GAPS`. Never use a prior client as defaults.

## Inputs — ask if missing

Parse the user message first. **Do not start** until required fields are known. Do not invent URLs, SSH targets, or site names.

1. Use any field already provided.
2. **Required and missing** — ask in one message (only missing required fields).
3. **Optional and missing** — ask once in the same message, each with skip/default below.
4. After answers or skips, proceed. Do not re-ask fields already given.

| Input | Required | Skip / default |
|-------|----------|----------------|
| `SSH_TARGET` | Yes unless already SSH’d | Current shell if already connected |
| `SITE_NAME` / client | Yes | Derive from URL / hostname only if the user agrees when asked |
| `SITE_URL` | Yes | — |
| `CHILD_THEME_PATH` | No | Discover via `wp theme list` |
| `BRAND_TOKENS` | No | Empty — skip token-literal check |
| `KNOWN_CONTENT_GAPS` | No | Empty list — never invent |
| `FIGMA_URL` | No | Skip Figma step |
| `OUTPUT_DIR` | No | `<project-root>/reports/<slug>-code-qa/` on the **local** machine |

`<project-root>` = Cursor workspace / git root on the machine writing the report. Never `~/reports`. Never write reports onto Kinsta. Honor an explicit `OUTPUT_DIR`.

**Known content gaps:** Note if still present. Do **not** flag as new bugs.

## Read-only guardrails (non-negotiable)

- Do **not** modify, delete, move, or write any files on the server.
- Do **not** run WP-CLI that alters data (`wp option update`, `wp post update`, `wp plugin activate/deactivate/install`, `wp cache flush`, `wp db *`, etc.).
- Do **not** install packages on the server (including npm). If tooling is missing, skip — never work around by installing.
- Do **not** enable `WP_DEBUG` / `WP_DEBUG_LOG` — only report what is already there.
- **Protected (read for context only, never write):** `wp-config.php`, `wp-login.php`, `wp-cron.php`, `xmlrpc.php`, root `index.php`, `wp-includes/`, `wp-admin/`, `wp-content/themes/Divi/` (parent), `.htaccess`, nginx configs, `php.ini`.
- **Max Mega Menu:** Never touch `/wp-content/uploads/maxmegamenu/style.css`. If custom CSS is there, flag migrate to child `theme.css` (or equivalent).

Exact commands: [checks.md](checks.md). Report: [report-template.md](report-template.md).

## Workflow checklist

```
QA Progress:
- [ ] 0 Environment recon
- [ ] 1 Theme structure
- [ ] 2 CSS practices
- [ ] 3 Code hygiene grep
- [ ] 4 PHP, security & agency standards
- [ ] 5 Plugin audit
- [ ] 6 Error log review
- [ ] 7 Maintainability & naming
- [ ] 8 Figma design review (only if FIGMA_URL)
- [ ] Report delivered
```

### 0 — Environment recon

Report before later steps: `whoami` / `id` / `pwd`; WP-CLI (`wp --info`); child theme path; `WP_DEBUG` / `WP_DEBUG_LOG` (read `wp-config.php` for context only); `debug.log` path/size.

If WP-CLI is absent, note which steps degrade to filesystem-only and continue.

### 1 — Theme structure

Confirm a **child theme** is active, not parent Divi edited directly.

- Active theme + parent via `wp theme list` (or `style.css` headers)
- Spot-check parent Divi for unexpected local edits — do not modify
- Map CSS: child `style.css` / `theme.css` / `assets/`, Divi Theme Options Custom CSS, Customizer `custom_css`
- Map PHP: `functions.php`, `functions/` includes, dedicated custom plugin, or scattered elsewhere
- Healthy shape: thin `style.css` header + `functions.php` + `functions/` + `assets/`
- Custom CSS/JS/PHP in plugin files or parent `Divi/` → flag migrate to child theme

### 2 — CSS practices

See [checks.md](checks.md):

1. **Centralization** — child stylesheet / Customizer over per-module Divi Custom CSS. Count non-empty module `custom_css`.
2. **Tokens** — for each `BRAND_TOKENS` value, literals vs custom properties / utilities. Skip if no tokens provided.
3. **`!important`** — count per file; heavy use is Coaching Note unless masking a real conflict (Developer Action).
4. **Media queries** — flag redundant `@media` that only duplicates Divi responsive controls.
5. **Prohibited Divi numbered classes** (`.et_pb_section_0`, `.et_pb_row_12`, …) — always **Developer Action**.
6. **CSS classes:** never `fsm_`. Flag `.fsm_*` as Developer Action; require semantic names (e.g. `.service-grid-card`).
7. **Swiper:** if Swiper is enqueued or markup/classes appear (`swiper`, `swiper-button-next/prev`, `swiper-pagination`) **and** custom arrow styles exist, require default, `:hover`, `:focus` / `:focus-visible`, `:active`, `[disabled]` / `.swiper-button-disabled`, and locked/end if used. Missing states → Moderate; unusable arrows → Critical.

### 3 — Code hygiene grep

Scope: child theme + custom/site-specific plugins. Exclude `node_modules`, `vendor`, minified bundles.

- Placeholders: `lorem ipsum`, `TODO`, `FIXME`, `XXX`, `coming soon`, `placeholder` (beyond `KNOWN_CONTENT_GAPS`)
- Debug: `console.log`, `var_dump`, `print_r`, `error_log`, `dd(`, `debugger`
- Secrets: `api_key`, `secret`, `password`, `token`, `Bearer `, `mysql://`, AWS-style keys

Map `KNOWN_CONTENT_GAPS` matches to Known/Expected.

### 4 — PHP, security & agency standards

Read-only greps (cap volume). Skip if `rg` missing.

- Unsanitized `$_GET` / `$_POST` / `$_REQUEST`; unescaped `echo` without nearby `esc_html` / `esc_attr` / `esc_url` (sample — do not false-positive every echo)
- Unprepared `$wpdb->query` / `get_results` string interpolation; missing `$wpdb->prepare`
- Privileged AJAX without nonce / `current_user_can`
- Scripts/styles printed in `wp_head` / `wp_footer` instead of `wp_enqueue_*`
- FacetWP: listing + filter bundled in one custom shortcode (`facetwp facet` + `facetwp template` in the same handler); `search_bar_hidden` / `show_search` workarounds
- Duplicate jQuery or Swiper (child + plugin + CDN). Mixed CDN + bundled → Moderate
- Hardcoded credentials vs `wp-config.php` constants (do not edit wp-config)

Do **not** implement fixes in this pass.

### 5 — Plugin audit

Read-only: `wp plugin list`; `wp plugin list --update=available`. Flag inactive, overlapping (cache/SEO/forms/security/builders), outdated. Without WP-CLI: list `wp-content/plugins/` and note limited confidence.

### 6 — Error log review

Only if `WP_DEBUG_LOG` is already on and the file exists. Tail, group unique messages, attribute child / custom plugin / parent / core. Never enable logging.

**PHP 8.1+ (Kinsta):** deprecations, notices, and fatals from the **child theme or custom plugins** → **Critical**. Core/parent noise → Coaching Note unless it breaks the page.

### 7 — Maintainability & naming

- **CSS/HTML:** never `fsm_`; semantic class names
- **PHP functions/hooks:** `fsm_` is **optional**. Shortcode handles may be `fsm_…` **or** a client/site prefix. Do **not** fail only because a shortcode lacks `fsm_`. PHP classes: descriptive PascalCase — no `FSM_` / `fsm_` on classes unless this is the FSM agency site
- Shortcodes: mandatory shortcode comment block
- Non-technical editability
- Unlabeled workarounds (`hack`, `quick fix`, `remove after launch`)

### 8 — Figma design review (optional)

**If `FIGMA_URL` empty:** `Not run — no Figma URL provided`.

**If provided:** load `/figma-design-to-code` first; parse `fileKey` / `nodeId`; MCP `get_variable_defs` then `get_design_context` / `get_screenshot`; compare to Step 2 tokens. If MCP unavailable, skip with reason — do not guess.

## Finding classification

| Severity | Use when |
|----------|----------|
| **Critical** | Secrets, parent theme edited, PHP 8.1+ fatals/deprecations in child/custom plugins, debug in production path, numbered Divi selectors, `.fsm_*` classes, unusable Swiper |
| **Moderate** | Scattered module CSS, inactive redundant plugins, missing shortcode docs, FacetWP bundled shortcodes, duplicate jQuery/Swiper, missing Swiper states, enqueue bypass |
| **Minor** | Token literals, `!important` habits, naming polish (not `fsm_` CSS) |

| Track | Use when |
|-------|----------|
| **Developer Action** | Code/config must change before launch |
| **Coaching Note** | Process/habit guidance |

## Explicit exclusions

- No Playwright, Lighthouse, or axe (use `browser-site-qa` on local/cloud).
- No inventing `KNOWN_CONTENT_GAPS`.
- No writes to Mega Menu generated CSS or protected paths.

## Output

1. Save a copy under local `OUTPUT_DIR` if writable (never onto Kinsta). Recommend client `.gitignore` include `reports/`.
2. One markdown report via [report-template.md](report-template.md).
3. **Priority Fix List (Developer Action only)** then **Coaching Notes**.
4. List skipped checks and why.

## Related

- Team install: [TEAM-INSTALL.md](TEAM-INSTALL.md)
- Companion: `browser-site-qa` (local or Cursor cloud only)
