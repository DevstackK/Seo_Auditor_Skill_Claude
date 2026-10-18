# Security & Quality Checks

This repo contains a Claude Code skill: `SKILL.md`, a Markdown file of instructions Claude follows, plus documentation. It has no application code, dependencies, or build step.

## Trivy scan

Run against the full repository with [Trivy](https://trivy.dev/) ([`fs` scanner](https://trivy.dev/latest/docs/target/filesystem/), vulnerability + secret + misconfiguration modes):

```
trivy fs --scanners vuln,secret,misconfig .
```

**Result: clean.**

```
┌────────┬──────┬─────────────────┬─────────┬───────────────────┐
│ Target │ Type │ Vulnerabilities │ Secrets │ Misconfigurations │
├────────┼──────┼─────────────────┼─────────┼───────────────────┤
│   -    │  -   │        -        │    -    │         -         │
└────────┴──────┴─────────────────┴─────────┴───────────────────┘
```

- **Secrets:** 0 found — no API keys, tokens, or credentials anywhere in the repo.
- **Vulnerabilities / misconfigurations:** no scannable targets exist (no `package.json`, lockfile, Dockerfile, or IaC config) — there's nothing here with dependencies or a config format Trivy checks, which is expected for a documentation-only skill repo.

Scanned with Trivy v0.75.0 on 2026-10-18.

## Why there's no test suite

This repo is `SKILL.md` + `README.md` + `LICENSE` — Markdown instructions, not executable code. There's nothing for Vitest (or any test runner) to run against. The skill's actual *behavior* (its curl-based crawlability checks, JSON-LD validation logic, etc.) has been exercised directly against a real production site ([kloudkoach.com](https://kloudkoach.com)) across multiple live runs during development, each one verified against the raw HTML/API responses rather than assumed — see the skill's own "Known limitations" section in `seo-crawler-audit/SKILL.md` for what's been validated versus what's still an estimate.

If this skill ever grows a companion script (e.g. a wrapper CLI) rather than staying pure-instructions, that would be the point to add a real Vitest/Jest suite against it.
