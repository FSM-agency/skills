# wp-child-theme-qa — Team install

Gives every teammate the same SSH / child-theme hygiene workflow. Invoke by asking for a child theme QA / code hygiene pass. The skill asks for any missing inputs — do not paste a filled prompt template.

Pairs with **browser-site-qa** (live URL). Browser QA runs **locally or on a Cursor cloud sandbox** — never on Kinsta (no npm).

You must be SSH’d into the client **development** environment, or provide `SSH_TARGET`, before the code pass can run.

## Option A — Cursor personal skills

```bash
cp -R wp-child-theme-qa ~/.cursor/skills/wp-child-theme-qa
```

Restart Cursor or start a new Agent chat.

## Option B — Project skill (client repo)

```bash
mkdir -p .cursor/skills
cp -R wp-child-theme-qa .cursor/skills/wp-child-theme-qa
```

## Option C — Marketplace / team plugin

Install `fsm-website-qa-skills` from the FSM skills marketplace. Keep `SKILL.md` at the skill root (`skills/wp-child-theme-qa/`).

## Package

```
wp-child-theme-qa/
├── SKILL.md
├── checks.md
├── report-template.md
└── TEAM-INSTALL.md
```

## Artifacts

If saved, default to `<project-root>/reports/<slug>-code-qa/` on the **local** machine running the agent. Never `~/reports`. Never write reports onto the Kinsta server.

Add `reports/` to the **client project** `.gitignore`.
