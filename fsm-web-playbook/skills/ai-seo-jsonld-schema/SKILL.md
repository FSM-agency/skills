---
name: ai-seo-jsonld-schema
description: >-
  Audits and improves JSON-LD for one target page by inspecting its live schema,
  filling supported gaps from content on the provided site, and extending the
  existing Yoast or All in One SEO graph through the WordPress child theme.
  Use when asked for JSON-LD, structured data, schema markup, AI SEO schema,
  Yoast schema, AIOSEO schema, or a validator.schema.org handoff. Requires a
  target page URL and general direction or starting point.
---

# AI SEO JSON-LD Schema

Improve the structured data for **one target page** without inventing claims or
creating a competing graph. The requested direction is the starting priority;
also check obvious gaps for the page type and site industry.

Read [WORKFLOW.md](WORKFLOW.md), [PAGE-TYPES.md](PAGE-TYPES.md), and
[CODE-PATTERNS.md](CODE-PATTERNS.md) before editing.

## Inputs — ask if missing

Parse the user message first. Do not start the audit or implementation until both
required inputs are known. Ask for all missing required inputs in one message.
Do not invent URLs, client names, facts, or task direction.

| Input | Required | Default |
|-------|----------|---------|
| `TARGET_PAGE_URL` | Yes | None; must be a canonical live or staging page |
| `DIRECTION` | Yes | None; a short task note, type hint, or starting point is enough |
| `SITE_NAME` | No | Derive from the provided site only when unambiguous |
| `SEO_PLUGIN` | No | Detect Yoast or AIOSEO; ask if ambiguous |
| `ALLOW_EXTERNAL_RESEARCH` | No | `false` |

Examples of sufficient direction: “add FAQ schema from the accordion,” “improve
the contact page’s LocalBusiness data,” or an AI SEO calendar task note.

## Non-negotiable rules

### Content fidelity

- Mark up only accurate content visible on the target page or clearly supported
  elsewhere on the **provided site**.
- Target page first; then inspect relevant same-site pages such as Home, About,
  Contact, location pages, footer links, and the site’s existing schema graph.
- Write concise schema values from existing on-site content when needed. Preserve
  meaning; do not add marketing claims that the source does not support.
- Never invent reviews, ratings, prices, availability, hours, addresses,
  credentials, awards, founding dates, relationships, or social profiles.
- Prefer fewer complete and accurate properties over many weak properties.
- Use the most specific applicable Schema.org type that the site supports.
- Schema must describe content available to a visitor. Do not use markup as a
  hidden place to publish new claims.

### External research gate

The target site is an allowed source; the wider web is not.

- Do **not** use search engines, directories, knowledge panels, social networks,
  Wikipedia, competitor sites, or other external sources unless the user gives
  explicit permission in this conversation.
- Official Schema.org, Google Search Central, and active SEO-plugin developer
  documentation may be consulted for implementation rules; they are standards
  references, not sources for client facts.
- If external research is allowed, record every adopted fact with its source URL
  in the handoff. Do not silently blend external and on-site content.
- If external data would help but permission is absent, continue with on-site
  evidence and list the unresolved item under **Needs human / off-site data**.

### WordPress implementation

- All custom schema PHP belongs in the active child theme’s
  `functions/schema.php`.
- If the file is absent, create it and include it once from `functions.php`.
- Never put schema functions in `functions/theme.php`, make one file per page,
  edit a plugin, edit the Divi parent theme, or edit WordPress core.
- Extend the active SEO plugin’s existing `@graph`; do not print a duplicate
  standalone JSON-LD graph for entities already managed by that plugin.
- Keep stable `@id` values and connect nodes using type-appropriate references.
  Use `mainEntityOfPage` when an entity is the page’s primary subject. For
  supporting entities, prefer relationships on the WebPage such as `about` or
  `mentions`. Use `hasPart` only for a `CreativeWork` that is genuinely part of
  the page, and do not apply `isPartOf` outside its Schema.org domain.
- Scope every change to the target page using canonical URL or a stable
  WordPress conditional. Do not accidentally change the whole site.
- Follow WordPress coding, sanitization, escaping, and child-theme standards.
- Preserve unrelated existing code in `schema.php`.

### Safety

- Treat the environment as production unless the user says otherwise.
- Before database writes, WP-CLI mutations, cache flushes, deployments, or live
  publishing, state the risk and obtain confirmation. File-only child-theme work
  follows the normal repository workflow.
- Do not change WordPress settings merely to select a schema type when a safe
  graph filter can implement the requested result.

## Required workflow

Follow [WORKFLOW.md](WORKFLOW.md) in order:

1. Confirm URL and direction.
2. Fetch the live page and capture all existing JSON-LD before coding.
3. Detect Yoast or AIOSEO and identify existing node IDs and relationships.
4. Classify page type and site industry from on-site evidence.
5. Build a gap map using [PAGE-TYPES.md](PAGE-TYPES.md).
6. Fill supported gaps from on-site content; gate all external research.
7. Implement with the patterns in [CODE-PATTERNS.md](CODE-PATTERNS.md).
8. Validate PHP and inspect the resulting live output after deployment.
9. Deliver the validator link and evidence summary.

If the repository change cannot be deployed by the agent, do not claim that the
live output contains it. Report code-level validation as complete and label the
post-deployment live check as pending.

## Required handoff

Always include:

- target URL and direction;
- detected SEO plugin and hooks changed;
- types/properties added, changed, or deliberately left alone;
- on-site source pages used;
- external source URLs used, or `External sources: none`;
- before/after graph highlights, clearly distinguishing deployed output from
  local expected output;
- unresolved gaps under **Needs human / off-site data**;
- this developer link with the page URL percent-encoded:
  `https://validator.schema.org/#url=<encoded-target-url>`;
- a stale-cache or deployment note when live output cannot yet reflect the code.

Schema Markup Validator checks Schema.org vocabulary. When the type targets a
Google rich result, also recommend Google’s Rich Results Test for Google-specific
eligibility; do not promise a rich result.
