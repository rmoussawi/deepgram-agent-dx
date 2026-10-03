# Findings

> **Status: draft.** Covers Conditions 0, A, B, and C1. Condition C (docs MCP) is pending a rerun because
> Deepgram's docs MCP server was down; sections marked **[C2]** will be updated if it runs.

## Summary
A coding agent built a working Deepgram meeting-summary tool in every condition with a key, with zero human interventions, in 37 to 75 seconds. The interesting differences weren't in whether it worked, but in **how current, how correct, and how honest the result was.** 

Without docs, the agent relied on prior knowledge and used a deprecated parameter without knowing it. 

With docs, it used the current API and produced visibly better speaker labels, but took about twice as long, and it misread an API response and told the developer something false about Deepgram. 

When the docs MCP server failed loudly, the agent recovered in seconds, but its
docs discovery degraded.

## Results
| Condition | Success | Wall time | First successful call | Interventions | Deprecated usage | Summarization via | Integration quality |
|---|---|---|---|---|---|---|---|
| 0: No key | Deferred | 34s | n/a | 0 | n/a | n/a | Handoff 2/4 |
| A: Baseline | Y | 37s | 12s | 0 | Yes (`diarize=true`) | `/v1/listen` `summarize=v2` | 4/5 |
| B: llms.txt | Y | 70s | 29s | 0 | No | `/v1/read` (after misreading the response) | 5/5 |
| C1: Docs MCP, server down | Y | 75s | 49s | 0 | No | `/v1/read` (summarization page not found) | 5/5 |
| C2: Docs MCP **[C2]** | | | | | | | |

Full data: [`experiments/agent-dx/results.md`](experiments/agent-dx/results.md). Run logs: [`experiments/agent-dx/runs/`](experiments/agent-dx/runs/).

## Hypotheses
| # | Hypothesis | Result | Evidence |
|---|---|---|---|
| 1 | Docs access reduces time to a working build and errors | **Rejected** | Docs runs were slower (70s, 75s vs. 37s). Errors didn't drop: B misread a response and drew a false conclusion. Docs improved *currency*, not speed. **[C2]** |
| 2 | Without docs, the agent picks up outdated patterns | **Supported, by a different mechanism** | A used the deprecated `diarize=true`. It did no web search at all; the stale pattern came from the model's prior knowledge |
| 3 | Without a key, the agent's handoff instructions are incomplete | **Supported, but reframed** | Handoff scored 2/4 (no signup or key-creation steps). More important: the agent didn't stop. It built untested code and deferred verification to the human |
| 4 | The agent uses a separate LLM for summarization | **Rejected** | Every run used Deepgram's own summarization. But in A, the agent recommended switching to another LLM for better summaries |

## Key findings

### 1. Prior knowledge goes stale silently
In Condition A, the agent consulted no docs and finished in 37 seconds. It worked, but it used `diarize=true`, which Deepgram's Diarization page marks deprecated in favor of `diarize_model`. Nothing told the agent or the developer: in a later controlled test, requests using `diarize=true` returned no warnings. 

**Fast and working is not the same as current.** For a well-known API, the agent experience is set by the model's training data first, and docs only matter if the agent decides to look.

### 2. Docs made the agent current, and the developer would see the difference
With llms.txt (B), the agent read the Diarization page and used `diarize_model=latest`. Same model, same clip: the transcription was word-for-word identical, but speaker attribution was clearly better in B. Several replies that A assigned to the wrong speaker were correct in B. 

A developer using A's tool would judge Deepgram's diarization by the older model without knowing a better one was one parameter away. *(One clip; B's code also smooths one-word speaker flips, which can't explain most of the difference.)*

**Cost:** roughly double the time (70s vs. 37s; 29s vs. 12s to first call) for four docs fetches.

### 3. A misread response became a false claim about the product
In B, the agent checked for the summary at the top level of the response. Deepgram returns it under `results.summary`, and the response metadata even showed summarization had run. The agent concluded summarization doesn't work with nova-3, built a workaround through `/v1/read`, and **wrote the false claim into the README.**

A controlled test confirmed the API was fine: four requests varying the diarizer and `language` setting, including B's exact settings, all returned a summary. 

The page itself is correct: its response example shows `summary` inside `results`. But the agent read a condensed version of the page that kept only "same level as `channels`" and dropped the example. Docs are increasingly read through summaries like this, so exact paths need to be in the prose, not only in examples. 

**When agents misread docs, developers inherit the mistake as a statement about the product.**

### 4. Loud errors work; agents recover
In C1, the docs MCP server returned an explicit error ("Failed to fetch from FAI chat service"). The agent recognized the outage within a second and switched to fetching pages directly. 

A clear error was handled well; the cost of the outage showed up later, in what the agent couldn't find (finding 5).

### 5. Agent-facing infrastructure is now production infrastructure
With no hint in the prompt, the C1 agent went to the docs MCP server first. It was down at 8:17 AM ET and still down at 8:45 AM and 12:33 PM ET, more than four hours. 

Without search, the agent guessed page URLs, never found the Summarization page, and told the developer that Deepgram offers summarization only for text. Both B and C1 ended up with an inaccurate claim about Deepgram in their README, for different reasons.

### 6. Agents defer when they can't verify
Without a key (Condition 0), the agent didn't stop to ask. It built the full tool in 34 seconds, checked for the key only at the very end, and handed verification to the human. It was honest about what it hadn't tested and never faked a result. But nothing let it confirm its integration before a human signed up and created a key.

### 7. Feature-by-model compatibility isn't stated anywhere an agent looks
In Condition 0, the agent couldn't confirm whether nova-3 supports summarization: the Summarization page says "Nova," which the Models page labels as legacy, and the SDK's type signature lists models and features separately. 

In B, the agent asked the Models page the same question and got no answer. Code examples also disagree by language (JavaScript and Java use `nova-3`; Python and Go use `nova-2`).

### 8. Agents evaluate products, not just integrate them
In A, the agent called Deepgram's summarizer "fairly literal" and suggested sending the transcript to another LLM instead. 

Agents act as reviewers and advisors to developers, and can steer them away from a feature.

## What I'd do next
See [`PROPOSAL.md`](PROPOSAL.md). <!-- Will pick the one fix with the strongest evidence. Candidates:
     deprecation warnings in API responses (1, plus the controlled test); exact response paths stated in prose, not only in examples (3); explicit feature-by-model compatibility in docs and SDK (7); a sandbox or test key so agents can verify before onboarding (6); monitoring agent-facing services like production APIs (5). -->

## Limitations
- **One run per condition**, one agent (Claude Code), one model (Claude Sonnet 5.5), one task, one 2-minute clip. These are signals, not statistics.

- **Condition C not yet administered:** the docs MCP server was down. **[C2]**

- **Setup differences, documented in the run logs:** Condition 0 ran in the Claude desktop app with the SDK preinstalled; A-C ran in the CLI with the SDK uninstalled. `requests` and `httpx` were installed for A-C, which may have shaped the agents' choice of raw HTTP.

- **My first read of B was wrong.** I initially logged B as a silent API failure, based on the agent's own account. The controlled test showed the agent misread the response. Agent claims need independent checks.

- **Agents read docs through a summarizing fetch tool**, so what they "read" was a condensed version of each page.

- **Runs happened on different days**, so a docs or API change between runs can't be fully ruled out.
