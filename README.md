# Custom Claude Code & Agent Skills

A collection of custom [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) skills built for **application security testing (DAST triage & authenticated pentesting)** and **end-to-end QA test automation (Cypress)**.

Each skill follows the `SKILL.md` standard (YAML frontmatter + structured operational instructions and reference rubrics) so it can be dropped directly into `~/.claude/skills/` or `.agents/skills/`.

---

## Included Skills

| Skill | Category | Purpose |
| :--- | :--- | :--- |
| [`pentest-validator`](skills/pentest-validator/SKILL.md) | Security / AppSec | Triages and validates automated scanner alerts (OWASP ZAP, DAST webhooks) using safe differential HTTP probing (`'` vs `''` vs `\'`, benign XSS markers, scheme checks, CSP/cookie inspection) and outputs an evidence table + ready-to-file bug ticket. |
| [`authenticated-endpoint-pentest`](skills/authenticated-endpoint-pentest/SKILL.md) | Security / AppSec | Runs a breadth-first authenticated security sweep across web apps and SPAs: session bootstrap, JS bundle route extraction, IDOR/cross-tenant authz, privilege escalation, injection, CSRF, mass-assignment, and gate-only (`id:0`) mutation probing. |
| [`recording-to-cypress`](skills/recording-to-cypress/SKILL.md) | QA Automation | Converts noisy Chrome DevTools Recorder JSON exports into clean, idiomatic Cypress end-to-end specs — stripping recorder noise, enforcing `[data-testid]` selector hierarchy, reusing Page Objects, scrubbing credentials, and asserting functional outcomes. |
| [`cypress-comment-style`](skills/cypress-comment-style/SKILL.md) | QA Automation | Enforces high-signal code and comment hygiene across Cypress specs and Page Objects — outcome-driven `it()` titles, zero step narration, and DOM/selector gotchas documented once on the Page Object. |

---

## Repository Structure

```text
custom-skills/
├── README.md
├── LICENSE
└── skills/
    ├── pentest-validator/
    │   ├── SKILL.md
    │   └── references/
    │       ├── severity-rubric.md
    │       └── bug-template.md
    ├── authenticated-endpoint-pentest/
    │   ├── SKILL.md
    │   └── references/
    │       ├── methodology.md
    │       └── roe-and-safety.md
    ├── recording-to-cypress/
    │   └── SKILL.md
    └── cypress-comment-style/
        └── SKILL.md
```

---

## Installation & Usage

Clone the repository and symlink the skills you want into your `~/.claude/skills/` directory:

```bash
git clone https://github.com/Martined27/custom-skills.git ~/custom-skills
mkdir -p ~/.claude/skills

ln -sfn ~/custom-skills/skills/pentest-validator ~/.claude/skills/pentest-validator
ln -sfn ~/custom-skills/skills/authenticated-endpoint-pentest ~/.claude/skills/authenticated-endpoint-pentest
ln -sfn ~/custom-skills/skills/recording-to-cypress ~/.claude/skills/recording-to-cypress
ln -sfn ~/custom-skills/skills/cypress-comment-style ~/.claude/skills/cypress-comment-style
```

Then invoke any skill in Claude Code by name (e.g., `/pentest-validator`, `/authenticated-endpoint-pentest`, `/recording-to-cypress`).

---

## License

[MIT](LICENSE) © 2026 Martin De Leon
