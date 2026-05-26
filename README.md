# Compliance & Audit Skills for AI Coding Agents

A set of four [SKILL.md](https://www.agensi.io/learn/create-skill-md-from-scratch) skills that turn an AI coding agent into a pre-launch auditor. Each one runs a focused, framework-aware audit of a codebase and writes a single severity-graded report with `file:line` citations and concrete fixes.

Built to the open **SKILL.md** standard — they work with Claude Code, Cursor, Codex CLI, VS Code Copilot, Gemini CLI, and any compatible agent. No runtime dependencies; the skills are pure instruction sets.

## The skills

| Skill | What it audits | Output |
|-------|----------------|--------|
| [Legal, Security & Compliance Auditor](legal-security-compliance-auditor/) | GDPR · UK GDPR · CCPA/CPRA · data security · LLM/AI disclosure. Auto-detects target markets (incl. LGPD/PIPL/DPDPA). | `COMPLIANCE_AUDIT.md` |
| [Mobile Store Compliance Auditor](mobile-store-compliance-auditor/) | App Store & Google Play rejection causes — privacy manifest, permissions, ATT, billing, target SDK. | `MOBILE_STORE_AUDIT.md` |
| [License & Dependency Compliance Auditor](license-dependency-compliance-auditor/) | Open-source license risk scoped to your distribution model — copyleft, AGPL-in-SaaS, attribution, source-available. | `LICENSE_AUDIT.md` |
| [Accessibility Auditor](accessibility-auditor/) | WCAG 2.2 AA for web & mobile — names/labels, keyboard, contrast, focus, ADA / EAA / Section 508. | `ACCESSIBILITY_AUDIT.md` |

## How they work

Each skill follows the same shape:

1. **Detect** — framework, platform, ecosystem, and (where relevant) target markets or distribution model. Asks the user when signals are ambiguous.
2. **Discover** — map the surfaces that carry obligations (data models, permissions, dependencies, UI components, etc.).
3. **Audit** — load the relevant checklist(s) from `references/` and run them against the discovery map.
4. **Report** — write one severity-graded Markdown file: Critical / High / Medium / Low, each finding with location, the cited rule/criterion, and a concrete fix.

Detailed checklists live in each skill's `references/` folder and are loaded on demand to keep the main `SKILL.md` lean.

## Sample output

See [`samples/`](samples/) for a full example report from each skill, generated against realistic fictional projects:

- [Legal/privacy/security sample](samples/legal-security-compliance-auditor.sample.md)
- [Mobile store sample](samples/mobile-store-compliance-auditor.sample.md)
- [License/dependency sample](samples/license-dependency-compliance-auditor.sample.md)
- [Accessibility sample](samples/accessibility-auditor.sample.md)

## Install

Copy a skill folder into your agent's skills directory. For Claude Code:

```
~/.claude/skills/<skill-name>/        # global, all projects
<project>/.claude/skills/<skill-name>/  # project-local
```

Then ask naturally — e.g. *"run a privacy audit"*, *"check this app before App Store submission"*, *"audit our dependency licenses"*, *"do a WCAG audit"* — and the agent loads the matching skill.

## Disclaimer

These skills produce **engineering audits, not legal advice**. License interpretation, privacy obligations, store-review outcomes, and accessibility conformance are fact-specific and subject to change. For binding decisions, consult a qualified professional.
