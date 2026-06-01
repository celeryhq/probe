# Contributing to AgentEval

Thanks for your interest in improving AgentEval. This project is a family of
Claude Code skills plus natural-language probe configs — there's no build step
and no compiler, so contributing is mostly about clear writing and well-shaped
examples.

## Ways to contribute

- **New probe scenarios** — add example suites under `scenarios/` that show off a
  pattern (routing, multi-step flows, guardrails, visual checks). Keep them
  product-agnostic so anyone can adapt them.
- **Skill improvements** — refine the orchestration, browser-interaction,
  verification, or report skills in `.claude/skills/`.
- **Docs** — sharpen the README, fix typos, add a how-to.
- **Bug reports** — open an issue describing what you probed, what you expected,
  and what happened (a screenshot or report snippet helps).

## Ground rules

- **Never commit secrets.** Credentials live in `.env` (gitignored). Scenarios
  reference them only via `{{env:VAR_NAME}}` placeholders.
- **Keep examples generic.** Don't commit scenarios, reports, or screenshots tied
  to a real product or customer — use a fictional target (see
  `scenarios/example.yaml`).
- **Match the surrounding style.** Skills are written as plain, instructional
  Markdown; scenarios are plain-language YAML. Read a neighbor before adding.

## Submitting a change

1. Fork the repo and create a branch (`git checkout -b my-change`).
2. Make your change. If you touched a skill, dry-run a scenario through Claude
   Code to confirm it still behaves.
3. Open a pull request describing the change and why it helps.

## Questions

Open an issue — happy to help you get a probe running.
