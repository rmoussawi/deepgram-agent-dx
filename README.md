# Building with Deepgram as an AI Agent

A hands-on look at how well Deepgram works when the developer is an AI coding agent, not a human.

## Why this repo exists
I started my career as an Android and iOS developer, writing the code myself. Today I'm a senior product manager building AI products for satellite network operations, including a multi-agent workflow that investigates the root cause of link degradation across RF and weather data, work that network engineers have traditionally done by hand. Software development is going through the same shift: more and more of the building is done by agents working on a developer's behalf.

That changes who a developer platform is really serving. Increasingly, the first "developer" to try an API isn't a person but a coding agent. If the agent can't discover, integrate, and correctly use a product, the person may never get to evaluate it. And if the agent integrates it poorly, they'll judge the product by a worse version of itself.

My first job was measuring how competing networks actually performed for the people using them. This repo applies the same approach to a new kind of user. I give Claude Code a simple task (building a meeting-summary tool with Deepgram, a voice AI platform) under different documentation setups, and measure how far it gets on its own, how correctly it uses the platform, and what helps. A separate probe tests what the agent does when it has no API key, and another leaves the provider unnamed, to see which one the agent picks. 

For contrast, the friction log also includes a short account of my own experience as a human developer following one of Deepgram's tutorials. It points to a different kind of friction: the agents struggled with what was current and correct, while I struggled with what to expect and what to do next.

## The question
Can an AI coding agent discover Deepgram, integrate it, and use it correctly without a human stepping in?
And how much do Deepgram's agent-facing resources (llms.txt and the docs MCP server) help?

## Hypotheses
Written and committed before any runs, and kept unchanged, including where they turned out to be wrong.

1. **Docs access:** Giving the agent llms.txt or the docs MCP server will reduce time to a working build and the number of errors, compared with web search alone.

2. **Staleness:** With web search alone, the agent will pick up at least one outdated endpoint, model, or SDK pattern.

3. **Onboarding handoff:** Without an API key, the agent can't self-onboard. When it stops, its instructions to the human (where to sign up, how to create a key, where to put it) will be incomplete or point to outdated pages.

4. **Discovery:** The agent will reach for a separate LLM for summarization instead of finding Deepgram's own feature.

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
| Path                    | What it is                                                                                               | Status                                                                                                     |
| ----------------------- | -------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `experiments/agent-dx/` | Protocol, run logs, and results for an onboarding probe, documentation conditions A, B, C, and C-alt, four repeat runs of A, and a provider selection probe | Complete. Condition C not administered (docs search outage); C-alt tested Deepgram's other docs MCP server |
| `FRICTION_LOG.md`       | Timestamped friction points, from me as a human and from the agent                                       | Ongoing                                                                                                    |
| `FINDINGS.md`           | Analysis of the results                                                                                  | Final                                                                                                      |
| `PROPOSAL.md`           | One-page PRD for the fix the evidence best supports                                                      | For discussion                                                                                             |
| `01-agent-loop/`        | Minimal agent loop in plain Python, built to understand the mechanics                                    | Planned                                                                                                    |

## Key findings
*Overall interpretation, not tested directly:* **For agents, the common path is decided by what the model already knows. Deepgram's docs matter most wherever the product has changed since the model learned it. To help, they have to be both reachable and current.**

**1. Without docs, the agent built fast, but used a deprecated parameter, and nothing flagged it.** With no docs, the agent (Claude Code with Claude Sonnet 5.5) built a working tool in 37 seconds without fetching any documentation. But it used a deprecated diarization parameter, and the API returned no warning, so neither the agent nor the developer knew. In four repeat runs, it again fetched no docs and used the same deprecated parameter every time.

**2. Docs can make agents current, but they add time and have to be written for how agents read them.** Pointed to `llms.txt`, the agent used the current diarization setting instead of the deprecated one. But reading the docs made that run about twice as long (70 seconds, versus 37 with no docs), and after misreading a condensed version of one page, the agent told the developer a working feature was broken.

**3. Today, Deepgram's docs path for agents is fragmented and fragile.** Agents and people are pointed to two different docs MCP servers from two providers. The one the agent-facing docs point to failed at every check for more than a day while appearing healthy, and without it, the agent told the developer a feature didn't exist. The other required a sign-in the setup page doesn't mention, and once connected, it led the agent to one of Deepgram's own recipes using the deprecated parameter.

**4. With no provider named, the agent chose AssemblyAI in 5 of 5 runs.** Asked to "use whichever speech-to-text service you think is best," it chose AssemblyAI for transcription and speaker labels, and Claude for the summary, every time, without consulting any docs. In every run, its first command searched for a `DEEPGRAM` environment variable; Deepgram was never chosen.

**Recommendation: give agents one dependable docs MCP server.** That means one canonical server advertised everywhere, with its authentication documented; automated test searches on a regular schedule, since a basic connection check showed the server as healthy while every search failed; a fallback to `llms.txt` when search fails; and recipes kept current. Alongside it, measure share of agent choices, since the agent picked another provider in every run without a named provider. See [`PROPOSAL.md`](PROPOSAL.md).

Full analysis, all eleven findings, and limitations: [`FINDINGS.md`](FINDINGS.md). Data: [`results.md`](experiments/agent-dx/results.md).

## Reproduce it
See `experiments/agent-dx/PROTOCOL.md` for the exact prompt, conditions, and rules.

## How I worked
I used AI in two different roles. **Claude Code** (CLI, Claude Sonnet 5.5) was the agent under test: it received the task prompt and worked unassisted. Separately, I used **Claude** as a thinking partner to design the protocol, parse session logs, and draft these documents.

I ran every condition myself, ran each finished tool to check whether it worked, checked every score against Deepgram's live docs, and made the final call on each conclusion. That mattered: my first read of Condition B, based on the agent's own account, was wrong. A controlled test I designed and ran showed the agent had misread a correct API response, and I corrected the record in the commit history.
