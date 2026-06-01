# Security Policy

## Reporting a vulnerability

If you discover a security issue in AgentEval, please report it privately rather
than opening a public issue.

- Use GitHub's **[private vulnerability reporting](https://github.com/kdceleryhq/agenteval/security/advisories/new)**
  (Security → Report a vulnerability), or
- email **security@simpleai.dev**.

Please include a description of the issue, steps to reproduce, and the potential
impact. We'll acknowledge your report and keep you updated on the fix.

## Scope & handling credentials

AgentEval drives a real browser against a target agent using credentials you
supply. To keep those safe:

- Credentials and tokens belong in `.env`, which is **gitignored** — never commit
  them. Scenarios reference them only via `{{env:VAR_NAME}}` placeholders.
- Probe output (screenshots, videos, HTML reports) can capture whatever is on
  screen, including authenticated content. Treat the `results/` directory as
  sensitive and review before sharing a report publicly.
- Browser session/auth state (e.g. `auth.json`) is gitignored for the same
  reason.

## Supported versions

This is an actively developed project; fixes land on the default branch. Please
test against the latest `main` before reporting.
