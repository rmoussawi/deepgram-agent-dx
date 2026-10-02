# Building with Deepgram as an AI Agent

A hands-on look at how well Deepgram works when the developer is an AI coding agent, not a human.

## Why this repo exists
I started my career as an Android and iOS engineer. Today I'm a senior product manager building AI products for satellite network operations, including a multi-agent workflow that investigates the root cause of link degradation across RF and weather data to help operators meet their SLAs.

Increasingly, the first "developer" to try an API isn't a person but a coding agent working on someone's behalf. If the agent can't discover, integrate, and correctly use a product, the person may never get to evaluate it. And if the agent integrates it poorly, they'll judge the product by a worse version of itself.

My first job was measuring how competing networks actually performed for the people using them. This repo applies the same approach to a new kind of user. I give Claude Code a simple task (building a meeting-summary tool with Deepgram, a voice AI platform) under three different documentation setups, and measure how far it gets on its own, how correctly it uses the platform, and what helps. A separate probe tests what the agent does when it has no API key.

## The question
Can an AI coding agent discover Deepgram, integrate it, and use it correctly without a human stepping in?
And how much do Deepgram's agent-facing resources (llms.txt and the docs MCP server) help?

## Hypotheses
<!-- Written before the runs to be tracked against the results. -->
1. Docs access: Giving the agent llms.txt or the docs MCP server will reduce time to a working build and the number of errors, compared with web search alone.
2. Staleness: With web search alone, the agent will pick up at least one outdated endpoint, model, or SDK pattern.
3. Onboarding handoff: Without an API key, the agent can't self-onboard. When it stops, its instructions to the human (where to sign up, how to create a key, where to put it) will be incomplete or point to outdated pages.
4. Discovery: The agent will reach for a separate LLM for summarization instead of finding Deepgram's own feature.

## Success metrics
| Metric | Why it matters |
|---|---|
| Agent time to first successful call | The agent equivalent of time-to-value |
| Human interventions per run | How autonomous the agent can really be |
| Deprecated API, model, or SDK usage | Whether stale content is reaching agents |
| End-to-end task success (Y / N / Partial) | Whether the agent can deliver at all |
| Handoff quality (x/4, Condition 0 only) | How well the agent hands off to a human at the step it can't do itself |
| Integration quality (x/5, see protocol checklist) | Whether the agent used Deepgram correctly, which shapes how good the product looks to the developer |

## What's here
| Path | What it is | Status |
|---|---|---|
| `experiments/agent-dx/` | Protocol, run logs, and results for an onboarding probe plus three documentation conditions | In progress |
| `FRICTION_LOG.md` | Timestamped friction points, from me as a human and from the agent | Ongoing |
| `FINDINGS.md` | Analysis of the results | After runs |
| `PROPOSAL.md` | One-page PRD for the fix the evidence best supports | After runs |
| `01-agent-loop/` | Minimal agent loop in plain Python, built to understand the mechanics | Planned |

## Key findings
<!-- To be filled in after the runs and linked to FINDINGS.md. -->

## Reproduce it
See `experiments/agent-dx/PROTOCOL.md` for the exact prompt, conditions, and rules.

## How I worked
<!-- To be filled in the end. Which coding agent and model I used, and how I used it -->
