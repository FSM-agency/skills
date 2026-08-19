# Exact checks — wp-child-theme-qa

All commands below are **read-only**. Do not substitute write/update/delete variants. If a binary is missing (`wp`, `rg`), skip that check and say so.

Assume WordPress root is the current working directory after SSH (or set `WP_PATH` and use `--path="$WP_PATH"` on every `wp` call).

Ignore globs for codebase greps:

```text
--glob '!node_modules/**' --glob '!vendor/**' --glob '!**/min/**' --glob '!**/*.min.js' --glob '!**/*.min.css' --glob '!**/dist/**'
```

---

## 0 — Environment recon

```bash
whoami
id
pwd
which wp || true
wp --info 2>/dev/null || true
wp theme list --status=active --format=table 2>/dev/null || true
ls -la wp-content/themes/ 2>/dev/null || true
# WP_DEBUG / WP_DEBUG_LOG — read only (never edit):
rg -n "WP_DEBUG" wp-config.php 2>/dev/null || true
ls -lh wp-content/debug.log 2>/dev/null || true
```

Resolve `CHILD_THEME_PATH` from the active child theme stylesheet directory when not provided.

---

## 1 — Theme structure

```bash
wp theme list --format=table
wp theme get "$(wp theme list --status=active --field=name)" --format=json
head -n 40 "$CHILD_THEME_PATH/style.css"
find "$CHILD_THEME_PATH" -maxdepth 3 \( -name '*.php' -o -name '*.css' -o -name '*.js' \) \
  ! -path '*/node_modules/*' | sort
ls -la wp-content/themes/Divi/ | head -n 40
wp post list --post_type=custom_css --fields=ID,post_title,post_modified --format=table 2>/dev/null || true
ls -lh wp-content/uploads/maxmegamenu/style.css 2>/dev/null || true
wc -l wp-content/uploads/maxmegamenu/style.css 2>/dev/null || true
```

Map PHP: open `functions.php` and note `require`/`include` of `functions/` (or similar). Look for a site-specific plugin under `wp-content/plugins/` matching the site/client slug. Flag custom CSS/JS/PHP in parent `Divi/` or third-party plugin files (migrate to child theme).

---

## 2 — CSS practices

### Centralization (module Custom CSS)

```bash
wp post list --post_type=page --post_status=publish --fields=ID,post_title --format=csv | head -n 80
wp post get <ID> --field=post_content | rg -o 'custom_css"[^"]*"' | wc -l
rg -l 'custom_css' --glob '*.php' "$CHILD_THEME_PATH" || true
```

### Brand token literals

Skip this block if `BRAND_TOKENS` is empty. Substitute each provided hex/font for `<PRIMARY_HEX>` and `<FONT_PATTERN>`:

```bash
rg -n --ignore-case '<PRIMARY_HEX>' \
  --glob '!node_modules/**' --glob '!vendor/**' --glob '!**/*.min.css' \
  "$CHILD_THEME_PATH"
rg -n --ignore-case '<FONT_PATTERN>' \
  --glob '!node_modules/**' --glob '!**/*.min.css' \
  "$CHILD_THEME_PATH"
rg -n 'var\(--|--[a-zA-Z0-9_-]+\s*:' "$CHILD_THEME_PATH" --glob '*.css' --glob '!node_modules/**' | head -n 80 || true
```

### `!important` density

```bash
rg -n '!important' "$CHILD_THEME_PATH" --glob '*.css' --glob '!node_modules/**'
rg -c '!important' "$CHILD_THEME_PATH" --glob '*.css' --glob '!node_modules/**'
```

### Media queries

```bash
rg -n '@media' "$CHILD_THEME_PATH" --glob '*.css' --glob '!node_modules/**'
```

### Prohibited Divi numbered selectors

```bash
rg -n '\.et_pb_(section|row|column|module|text|image|blurb)_[0-9]+' \
  "$CHILD_THEME_PATH" \
  --glob '!node_modules/**' --glob '!**/*.min.css'
```

### Prohibited `fsm_` CSS classes

```bash
rg -n '\.fsm_' "$CHILD_THEME_PATH" --glob '*.css' --glob '!node_modules/**' --glob '!**/*.min.css'
rg -n 'class="[^"]*fsm_' "$CHILD_THEME_PATH" --glob '*.php' --glob '!node_modules/**' || true
```

### Swiper custom arrow states

If Swiper appears (enqueue, `swiper` class, or `swiper-button-`):

```bash
rg -n -i 'swiper' "$CHILD_THEME_PATH" --glob '!node_modules/**' | head -n 80
rg -n 'swiper-button-disabled|swiper-button-lock|:hover|:focus|:focus-visible|:active|\[disabled\]' \
  "$CHILD_THEME_PATH" --glob '*.css' --glob '!node_modules/**'
```

Require custom next/prev styles to cover default, hover, focus, active, disabled (and lock/end if used).

---

## 3 — Code hygiene grep

Set `SCAN_ROOTS` to `"$CHILD_THEME_PATH"` plus `wp-content/plugins/<custom-plugin>` only (not every third-party plugin unless a site-wide secret scan is requested).

### Placeholders

```bash
rg -n -i 'lorem ipsum|TODO|FIXME|\bXXX\b|coming soon|placeholder' \
  $SCAN_ROOTS \
  --glob '!node_modules/**' --glob '!vendor/**' --glob '!**/*.min.js' --glob '!**/*.min.css' --glob '!**/dist/**'
```

### Debug statements

```bash
rg -n 'console\.log\s*\(|var_dump\s*\(|print_r\s*\(|error_log\s*\(|\bdd\s*\(|\bdebugger\b' \
  $SCAN_ROOTS \
  --glob '!node_modules/**' --glob '!vendor/**' --glob '!**/*.min.js' --glob '!**/dist/**'
```

### Secrets / credentials patterns

```bash
rg -n -i 'api[_-]?key|secret[_-]?key|password\s*=|Bearer [A-Za-z0-9._-]+|mysql://|mysqli?://|AKIA[0-9A-Z]{16}|BEGIN (RSA |OPENSSH )?PRIVATE KEY' \
  $SCAN_ROOTS \
  --glob '!node_modules/**' --glob '!vendor/**' --glob '!**/*.min.js' --glob '!**/dist/**'
```

Filter documentation examples and `KNOWN_CONTENT_GAPS` into Known/Expected.

---

## 4 — PHP, security & agency standards

Cap samples (head). Do not false-positive every `echo`.

```bash
rg -n '\$_(GET|POST|REQUEST)\s*\[' $SCAN_ROOTS --glob '*.php' --glob '!node_modules/**' | head -n 60
rg -n 'echo\s+\$_' $SCAN_ROOTS --glob '*.php' --glob '!node_modules/**' | head -n 40
rg -n '\$wpdb->(query|get_results|get_var|get_row)\s*\(' $SCAN_ROOTS --glob '*.php' --glob '!node_modules/**' | head -n 40
rg -n 'wp_ajax_|admin_post_' $SCAN_ROOTS --glob '*.php' | head -n 40
rg -n 'check_ajax_referer|wp_verify_nonce|current_user_can' $SCAN_ROOTS --glob '*.php' | head -n 40
rg -n "add_action\s*\(\s*['\"]wp_(head|footer)['\"]" $SCAN_ROOTS --glob '*.php'
rg -n 'wp_enqueue_(script|style)' $SCAN_ROOTS --glob '*.php' | head -n 40
rg -n 'facetwp facet|facetwp template|search_bar_hidden|show_search' $SCAN_ROOTS --glob '*.php'
rg -n -i "wp_enqueue_script\s*\(\s*['\"]jquery|swiper" $SCAN_ROOTS --glob '*.php'
rg -n -i 'cdn\.jsdelivr|unpkg\.com|cdnjs\.cloudflare.com/ajax/libs/(jquery|swiper)' $SCAN_ROOTS --glob '!node_modules/**' | head -n 40
```

FacetWP: listing and filters must be **separate** shortcodes — not both `[facetwp facet]` and `[facetwp template]` inside one custom handler.

Duplicate jQuery/Swiper (enqueue + CDN) → Moderate.

---

## 5 — Plugin audit

```bash
wp plugin list --format=table
wp plugin list --update=available --format=table
wp plugin get <slug> --format=json
```

Flag overlap: caching, SEO, forms, security, page builders, image optimization.

Filesystem fallback:

```bash
ls -1 wp-content/plugins/
```

---

## 6 — Error log review

Only if logging is already enabled and the file exists:

```bash
ls -lh wp-content/debug.log
tail -n 200 wp-content/debug.log
tail -n 500 wp-content/debug.log | sed -E 's/^\[.*\] //' | sort | uniq -c | sort -rn | head -n 40
```

Attribute paths under the child theme, `themes/Divi`, `plugins/`, or core.

PHP 8.1+ deprecations / notices / fatals in **child theme or custom plugins** → Critical. Core/parent noise → Coaching Note unless page-breaking.

Do **not** create, truncate, or enable the log.

---

## 7 — Maintainability & naming

```bash
rg -n '^\s*function\s+' "$CHILD_THEME_PATH" --glob '*.php' --glob '!node_modules/**'
rg -n "add_shortcode\s*\(" "$CHILD_THEME_PATH" --glob '*.php'
rg -n -U '/\*\*[\s\S]{0,400}Shortcode:' "$CHILD_THEME_PATH" --glob '*.php' || true
rg -n '^\.[a-zA-Z]' "$CHILD_THEME_PATH" --glob '*.css' --glob '!node_modules/**' | head -n 80
rg -n '\.fsm_|class="[^"]*fsm_' "$CHILD_THEME_PATH" --glob '!node_modules/**' || true
wp post list --post_type=page --post_status=publish --fields=post_name,post_title --format=table | head -n 60
```

`fsm_` on PHP functions/hooks/shortcodes is **optional** (client prefix is fine). Do not fail for missing `fsm_` on shortcodes. Flag `.fsm_*` CSS. PHP classes: no `FSM_` / `fsm_` unless this is the FSM agency site.

```bash
rg -n -i 'workaround|quick fix|hack|temp fix|remove after|FIXME|HACK' \
  "$CHILD_THEME_PATH" --glob '!node_modules/**'
```

---

## 8 — Figma (when FIGMA_URL provided)

Not shell. Agent steps:

1. Load `/figma-design-to-code`.
2. Parse URL → `fileKey`, `nodeId` (`-` → `:`).
3. MCP: `get_variable_defs` → compare to `BRAND_TOKENS` and CSS from Step 2.
4. MCP: `get_design_context` / `get_screenshot` as needed.
5. If MCP auth fails or server unavailable → skip with reason.

---

## Severity / track reminders

- Secrets, parent edits, numbered Divi selectors, `.fsm_*` CSS, PHP 8.1+ child/custom-plugin log issues, PHP fatals → **Critical** + **Developer Action**
- Scattered module CSS, redundant plugins, missing shortcode docs, FacetWP bundled shortcodes, duplicate jQuery/Swiper, missing Swiper states → usually **Moderate**
- Token duplication, `!important` habits, PHP prefix preference → **Coaching Note** unless blocking
