# browser-site-qa — Team install

Gives every teammate the same browser/frontend QA workflow. Invoke by asking for a browser QA / frontend QA / Lighthouse pass. The skill asks for any missing inputs — do not paste a filled prompt template.

**Runtime:** local Cursor or a Cursor cloud sandbox with Node + npm. **Never on Kinsta** (no npm; do not install Playwright there). Pair SSH/code hygiene with `wp-child-theme-qa`.

## Option A — Cursor personal skills

```bash
cp -R browser-site-qa ~/.cursor/skills/browser-site-qa
```

Restart Cursor or start a new Agent chat.

## Option B — Project skill (client repo)

```bash
mkdir -p .cursor/skills
cp -R browser-site-qa .cursor/skills/browser-site-qa
```

## Option C — Marketplace / team plugin

Install `fsm-website-qa-skills` from the FSM skills marketplace. Keep `SKILL.md` at the skill root (`skills/browser-site-qa/`).

## Package

```
browser-site-qa/
├── SKILL.md
├── report-template.md
└── TEAM-INSTALL.md
```

## Artifacts

Default: `<project-root>/reports/<slug>-qa/` on the machine running Node (workspace / git root). Never `~/reports`. Never write reports onto Kinsta.

Add `reports/` to the **client project** `.gitignore` so Lighthouse, axe, screenshots, and throwaway audit projects are not committed.
