# Agent Context

Place files here that describe the agent under test. These inform eval generation and help the testing agent understand what to expect.

## What to include

- **Agent description** — what the agent does, its capabilities, personality
- **Skill files** — SKILL.md files that define the agent's behavior
- **Product docs** — feature specs, user guides, onboarding flows
- **System prompts** — the agent's instruction set (if shareable)
- **Example conversations** — sample interactions showing expected behavior

## How it's used

When generating eval scenarios, the testing agent reads these files to understand:
- What the agent should do (capabilities)
- What the agent should NOT do (boundaries)
- Expected tone, format, and quality standards
- Specific features to test (forms, widgets, flows)

## File naming

```
context/
  agent-overview.md          # High-level: what is this agent?
  skills/                    # Agent's skill files
    slide-designer.md
    ad-creative.md
    article-writer.md
  product/                   # Product context
    onboarding-flow.md
    feature-list.md
```
