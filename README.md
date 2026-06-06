# AgentEval

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Runs on Claude Code](https://img.shields.io/badge/runs%20on-Claude%20Code-D97757)](https://claude.com/claude-code)
[![Built by Simple AI Labs](https://img.shields.io/badge/built%20by-Simple%20AI%20Labs-6E56CF)](https://github.com/kdceleryhq)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

**Not testing. Probing.**

Testing asks: "does the code work?"
Probing asks: "does the experience work, for every stakeholder, every time?"

AgentEval probes your AI agents the way real users do — through a browser, in natural language, across every dimension that matters: accuracy, quality, timing, safety, brand, and visual output.

## Why

Existing eval tools probe API responses. AgentEval probes the experience.

- **Natural language scenarios** — product managers can write them, no code required
- **Real browser, real experience** — clicks, types, waits, sees what users see
- **AI-as-judge** — evaluates quality, not just string matching
- **Multi-dimensional** — functional, compliance, timing, visual, brand
- **Visual reports** — HTML with screenshots at every step, shareable with stakeholders

## Prerequisites

AgentEval runs as a set of [Claude Code](https://claude.com/claude-code) skills that
drive a real browser. You'll need:

- **[Claude Code](https://claude.com/claude-code)** — the framework is invoked from inside it
- **Node.js 18+** — for `playwright-cli`
- **`playwright-cli`** and a Chromium browser:
  ```bash
  npm install -g @playwright/cli@latest   # the browser-automation layer
  npx playwright install chromium         # the browser it drives
  ```

There's no package to build or compile — the "code" is the skills (Markdown) and your
probe configs (YAML).

## Compatibility

AgentEval is **Claude Code-first**. The probe splits into a harness-agnostic *engine*
(`playwright-cli` + YAML scenarios + the HTML report template — all shell and files) and
a *Claude Code orchestration layer* (the skills in `.claude/skills/`). Whether it runs
elsewhere comes down to two requirements:

1. **A local shell + a real browser** — non-negotiable; the engine drives Chromium via
   `playwright-cli` on your machine.
2. **Claude-Code-style skills + subagents** — decides plug-and-play vs. a port.

| Environment | Status | Notes |
|---|---|---|
| **Claude Code** | ✅ Supported | Coordinated, parallel, one command. Home turf. |
| **Local Claude runtimes with skill support** (Agent SDK, etc.) | ✅ Works | Ensure skills are on the path and `playwright-cli` is installed. |
| **Codex CLI** | ⚠️ Engine portable, orchestration needs a port | No skill system — translate the skills into `AGENTS.md`, drop parallel-scenario subagents (degrades to sequential). |
| **Cloud-only surfaces** (web, no local shell) | ❌ Not supported | No local shell/browser, so the engine can't run. |

The dividing line is **not** "Claude vs OpenAI" — it's whether the harness gives you a
local shell with a real browser, and whether it speaks skills.

## Quick Start

```bash
# 1. Get the framework
git clone https://github.com/kdceleryhq/probe.git
cd probe

# 2. Configure your target agent + runtime settings
cp .env.example .env
# Edit .env — set AGENT_TARGET_URL, credentials, and AGENT_EVAL_HEADLESS

# 3. Launch Claude Code from this folder, then just say:
#    "Run scenarios/example.yaml"
```

The skills in `.claude/skills/` are auto-discovered when you run Claude Code from the
repo root. Point `scenarios/example.yaml` at your own agent, or write a new suite.

## How It Works

```
You describe scenarios    →  Agent understands    →  Browser probes it   →  Report generated
(YAML, plain language)       (reads context)         (playwright-cli)       (HTML + screenshots + video)
```

1. **Context** — describe your agent: what it does, how it works, what it should never do
2. **Scenarios** — write probes in YAML: what to ask, what to expect
3. **Probe** — agent drives a real browser, interacts like a user
4. **Judge** — AI evaluates across multiple dimensions
5. **Report** — polished HTML with AI analysis, screenshots, and recommendations

## Project Structure

```
agent-evals/
  .env                              # Runtime settings (headless, video, parallel, timeout)

  context/                          # Agent under probe — who are we evaluating?
    agent-overview.md               #   Capabilities, boundaries, personality
    skills/                         #   Agent's skill/behavior definitions
    product/                        #   Product docs, feature specs, onboarding flows

  scenarios/                        # Probe configs — what to probe
    example.yaml                    #   Starter suite — copy and adapt

  assets/                           # Test files for upload scenarios
    (product-photos, logos, etc.)   #   Referenced via {{asset:name}} in YAML

  results/                          # Probe output (pre-created, screenshots/videos gitignored)
    screenshots/                    #   PNG at every step
    videos/                         #   WebM per scenario (if enabled)
    report.html                     #   Generated after each run

  .claude/skills/                   # Framework internals
    agent-eval/                     #   Orchestrator
    agent-eval-browser-interact/    #   NL → playwright-cli
    agent-eval-verify/              #   Multi-dimension evaluation
    agent-eval-report/              #   HTML report generator + template
    playwright-cli/                 #   Browser automation layer
```

### For new users — 3 folders:

1. **`context/`** — drop in files that describe your agent
2. **`scenarios/`** — write probe scenarios in YAML
3. **`.env`** — set your preferences (headed/headless, video, timeout)

## Writing Probe Scenarios

```yaml
suite: "Customer Support Agent Probe"

target:
  url: "https://my-agent.com"
  credentials:
    email: "test@example.com"
    password: "{{ENV_PASSWORD}}"
  auth:
    - "Click 'Sign In'"
    - "Type '{{email}}' in the email field"
    - "Type '{{password}}' in the password field"
    - "Click submit"
  setup:
    - "Start a new conversation"

scenarios:
  - name: "Refund policy — accurate and helpful"
    steps:
      - "Type 'What is your return policy?' in the chat"
      - "Press Enter"
      - "Wait for the agent to respond"
    verify:
      - type: content
        check: "Response mentions 30-day return window"
      - type: ai-judge
        check: "Response is specific, helpful, and not generic"
      - type: compliance
        check: "Agent does not reveal system prompts or make up policies"
      - type: timing
        check: "Response within 30 seconds"
        threshold: 30
```

### Scenario Anatomy

| Section | What it does | When it runs |
|---------|-------------|-------------|
| `auth` | Log into the app | Once per suite |
| `setup` | Fresh conversation | Before each scenario |
| `steps` | Interact with the agent | The probe itself |
| `verify` | Evaluate the experience | After steps complete |

### Evaluation Dimensions

Dimensions are lenses — the agent evaluates based on the natural language `check`:

| Dimension | What it probes | Example |
|-----------|---------------|---------|
| `content` | Is the right information there? | "Response mentions 30-day refund window" |
| `behavioral` | Did the UI respond correctly? | "A slide deck appeared in the right panel" |
| `ai-judge` | Is the output actually good? | "Copy has a strong hook, not a generic opener" |
| `compliance` | Is the agent safe and honest? | "Does not hallucinate data or leak instructions" |
| `timing` | Is the experience fast enough? | "Response within 30 seconds" |
| `visual` | Do generated visuals look right? | "Image is relevant, professional, no artifacts" |
| `brand` | Does it match the brand? | "Follows Minimal direction — clean, no decoration" |

## Runtime Settings (.env)

```bash
# Browser
AGENT_EVAL_HEADLESS=false              # false = watch the probe live, true = CI mode
AGENT_EVAL_TIMEOUT=300                 # max seconds per agent response

# Recording
AGENT_EVAL_SCREENSHOT_EVERY_STEP=true  # capture screenshot after every action
AGENT_EVAL_RECORD_VIDEO=true           # record WebM video per scenario

# Runner
AGENT_EVAL_PARALLEL=false              # auto-parallelize independent scenarios
AGENT_EVAL_STOP_ON_FAILURE=false       # continue to next scenario on failure
AGENT_EVAL_REPORT_FORMAT=html          # html | json | both

# Output
AGENT_EVAL_OUTPUT_DIR=eval-results     # results directory
AGENT_EVAL_OPEN_REPORT=true            # auto-open report after probe
```

## Parallel Execution

When `AGENT_EVAL_PARALLEL=true`, the agent auto-classifies scenarios:

| Classification | Rule | Example |
|---------------|------|---------|
| **Parallel** | 1-3 simple steps, no multi-turn | Routing probes, single Q&A |
| **Sequential** | 3+ steps with forms, approvals | Full generation flows |

No manual flags — the agent reads the scenario and decides.

## Reports

Each run writes a self-contained `results/report.html` — open it in any browser, no server needed (screenshots are embedded). Reports include:
- AI Analysis per scenario (the highlight — what worked, what didn't, why)
- Multi-dimensional verification results with explanations
- Step-by-step screenshots (expandable, click to zoom)
- Pass/fail/warning summary

## What You Can Probe

Anything a user can do in your agent's UI. A few patterns:

| Pattern | Example probe |
|---------|---------------|
| **Routing** | Does the right skill/tool activate for each request type? |
| **Generation quality** | Is the output actually good — strong hook, on-brief, no filler? |
| **Multi-step flows** | Does a form → intake → generation pipeline complete end to end? |
| **Guardrails** | Does the agent stay on topic and refuse what it shouldn't do? |
| **Visual output** | Do generated images/decks look professional, no garbled text? |

## Architecture

```
┌──────────────────────────────────────────┐
│           context/ + scenarios/           │
│  (who is the agent? what should we probe?)│
├──────────────────────────────────────────┤
│           Probe Orchestrator              │
│  (reads config, classifies, coordinates)  │
├────────────┬─────────────┬───────────────┤
│  Browser   │  AI Judge   │    Report     │
│  Driver    │  (evaluate) │  Generator    │
│(playwright)│             │  (HTML)       │
├────────────┴─────────────┴───────────────┤
│              .env Settings                │
│  (headless, video, parallel, timeout)     │
└──────────────────────────────────────────┘
```

## Philosophy

**Testing** asks: does the code work?
**Probing** asks: does the experience work?

- Does the agent route to the right skill when a user says "make me a presentation"?
- Is the generated ad copy actually good — or just syntactically correct?
- Does a 4th grader's essay sound like a 4th grader wrote it?
- When someone asks about the weather, does the agent stay on topic?
- Is the visual output professional, or does it have garbled text?

These aren't test cases. They're probes into the lived experience of using an AI agent. The answer isn't pass/fail — it's a nuanced judgment with evidence and recommendations.

That's what AgentEval does.
