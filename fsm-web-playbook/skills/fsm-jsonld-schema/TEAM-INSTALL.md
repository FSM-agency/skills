# fsm-jsonld-schema — Team install

Gives teammates the same page-specific JSON-LD workflow for WordPress sites using
Yoast SEO or All in One SEO.

Invoke it by asking for JSON-LD/schema work and provide:

1. the target page URL; and
2. general direction or a starting point.

Example:

```text
Improve JSON-LD for https://example.com/services/example/.
Start by adding Service schema and strengthening the visible FAQ.
Do not use external research.
```

The skill asks once for missing required inputs. It always inspects the live
graph, checks page-type and industry gaps, uses content from the provided site
first, and returns a Schema Markup Validator link.

## Option A — Cursor personal skill

```bash
cp -R fsm-jsonld-schema ~/.cursor/skills/fsm-jsonld-schema
```

Restart Cursor or start a new Agent chat.

## Option B — Project skill

```bash
mkdir -p .cursor/skills
cp -R fsm-jsonld-schema .cursor/skills/fsm-jsonld-schema
```

## Option C — Marketplace / team plugin

Install `fsm-web-playbook` from the FSM skills marketplace. Keep `SKILL.md` at:

```text
fsm-web-playbook/skills/fsm-jsonld-schema/SKILL.md
```

## Package

```text
fsm-jsonld-schema/
├── SKILL.md
├── WORKFLOW.md
├── CODE-PATTERNS.md
├── PAGE-TYPES.md
└── TEAM-INSTALL.md
```

## Operational notes

- Work in the client child-theme repository.
- Custom schema belongs in `functions/schema.php`.
- The agent must not edit Yoast, AIOSEO, WordPress core, or the Divi parent theme.
- External factual research is off by default. Grant permission explicitly when
  it is wanted.
- Deployment and cache mutations require separate confirmation when they affect
  production.
- The validator link is for a developer to check **after deployment**. Local code
  changes cannot alter live validator output until deployed.
