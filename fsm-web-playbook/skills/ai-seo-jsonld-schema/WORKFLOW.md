# JSON-LD workflow

Run the same sequence for every target page. A skipped step needs a reason.

```text
Schema Progress:
- [ ] 0 Inputs confirmed (URL + direction)
- [ ] 1 SEO plugin detected
- [ ] 2 Before: live JSON-LD captured
- [ ] 3 Page type + industry inferred
- [ ] 4 Gap map written (on-site sources named)
- [ ] 5 External research: skipped or approved + sourced
- [ ] 6 functions/schema.php implemented or updated
- [ ] 7 PHP/code checks passed
- [ ] 8 After: deployed live JSON-LD rechecked
- [ ] 9 Handoff: validator.schema.org link + change summary
```

## 0. Confirm inputs

Require `TARGET_PAGE_URL` and `DIRECTION`. Normalize the URL for comparison but
retain the user’s canonical URL for the validator handoff. Default
`ALLOW_EXTERNAL_RESEARCH` to false.

## 1. Detect the SEO graph owner

Use read-only evidence:

1. Inspect live HTML and JSON-LD script classes/comments.
2. Inspect the repository’s plugin list or integration code when available.
3. Use read-only WP-CLI only when already configured and safe.

Common signals:

- Yoast: `yoast-schema-graph`, `Yoast SEO`, IDs such as `#website` and
  `#organization`.
- AIOSEO: `aioseo-schema`, `All in One SEO`, graph output produced by AIOSEO.

If both plugins appear active or ownership is unclear, stop and ask. Never have
two plugins own the same graph.

## 2. Capture the live baseline

Fetch the target page before editing. Record:

- HTTP status, final URL, canonical, and robots state;
- every `application/ld+json` block;
- each node’s `@type`, `@id`, and relationships;
- duplicate or conflicting entities;
- validation errors visible from the graph itself.

Do not assume the pasted graph or repository mirrors production. The live page is
the baseline.

## 3. Gather the allowed content corpus

Use the target page first. Then follow same-site links only as needed:

- homepage and existing Organization/LocalBusiness node;
- About and Contact;
- relevant service, staff, location, author, or policy pages;
- footer business details and linked social profiles;
- WordPress post data available in the repository or via read-only APIs.

For each proposed property, note the source URL or local source. When sources
conflict, do not choose silently; report the conflict.

## 4. Classify and map gaps

Choose the page’s primary archetype and inspect the corresponding section in
[PAGE-TYPES.md](PAGE-TYPES.md). Infer industry only from on-site evidence.

Create a compact map:

| Candidate | Existing state | On-site evidence | Action |
|-----------|----------------|------------------|--------|
| Type/property | Missing/weak/complete | URL or code source | Add/merge/leave |

Address the user’s direction first, then obvious high-confidence gaps. Avoid
schema for its own sake.

## 5. Handle external information

If a useful property lacks on-site support:

1. Do not search externally by default.
2. Either omit it and record it under **Needs human / off-site data**, or ask for
   explicit permission when it materially affects the task.
3. With permission, prefer authoritative first-party sources.
4. Record the exact source URL for every external fact incorporated.

Implementation documentation may always be checked against official
Schema.org, Google, Yoast, and AIOSEO docs.

## 6. Implement in the child theme

Read [CODE-PATTERNS.md](CODE-PATTERNS.md).

- Reuse `functions/schema.php`; create it only when absent.
- Ensure `functions.php` includes it once, matching the theme’s include style.
- Use contextual, collision-resistant function names.
- Scope by canonical or stable WordPress conditional.
- Merge existing nodes by `@id` or recognized type rather than duplicating them.
- Reference existing Organization, WebSite, Person, and WebPage IDs.
- Use native WordPress values when they are the source of visible page data.
- Keep additions deterministic and server-rendered.

## 7. Verify code before deployment

At minimum:

- `php -l` every changed PHP file;
- inspect the diff for target scoping and accidental global changes;
- confirm the include cannot fatal when the file loads;
- confirm strings, URLs, and array shapes are safe;
- confirm required and recommended fields for the selected Google feature from
  current official documentation;
- confirm no duplicate/conflicting node was introduced.

Run repository-specific lint/tests when available.

## 8. Recheck deployed live output

After deployment, fetch the exact target URL again and compare the graph. If the
change is stale, the developer may load:

```text
<target-url>?kinsta-cache-cleared=all-cache
```

Do not run a broad cache flush without explicit confirmation. If deployment is
outside the current task, mark this step **pending after deploy** rather than
pretending the local code is live.

Construct the required handoff link:

```text
https://validator.schema.org/#url=<percent-encoded-target-url>
```

Schema.org validation does not guarantee Google rich-result eligibility. Use
Google’s Rich Results Test as an additional check for supported types.

## 9. Handoff template

```markdown
Target: <URL>
Direction: <starting point>
SEO plugin / hook: <plugin and filter>

Schema changes:
- <type/property> — source: <same-site URL>

External sources: none

Before / after:
- <deployed graph comparison, or local expected output + deployment pending>

Needs human / off-site data:
- None

Validate after deployment:
https://validator.schema.org/#url=<encoded URL>
```
