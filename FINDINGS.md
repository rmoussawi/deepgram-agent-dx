# Findings

> **Status: final.** Covers Conditions 0, A, B, C1, and C-alt. Condition C (the docs MCP server advertised in the agent-facing docs) could not be administered: its search was down throughout. C-alt tested Deepgram's other documented docs MCP server instead (Amendment 2).

## Summary
A coding agent built a working Deepgram meeting-summary tool in every condition with a key, with zero human interventions, in 37 to 75 seconds. The interesting differences weren't in whether it worked, but in **how current, how correct, and how honest the result was.** 

Without docs, the agent relied on prior knowledge and used a deprecated parameter without knowing it. 

With `llms.txt`, it used the current diarization setting instead of the deprecated one, but took about twice as long, and it misread an API response and told the developer something false about Deepgram. 

When the docs MCP server's search failed with a clear error, the agent recovered in seconds, but its docs discovery degraded.

With Deepgram's other docs MCP server, the agent found the right summarization approach in two searches, but followed one of Deepgram's own code recipes that used the deprecated parameter, ending up exactly where the no-docs baseline did. Across the study, Deepgram's docs gave agents two different MCP servers and three different pointers, and the one agents find first was down for more than a day without anyone noticing.

## Results
| Condition | Success | Wall time | First successful call | Interventions | Deprecated usage | Summarization via | Integration quality |
|---|---|---|---|---|---|---|---|
| 0: No key | Deferred | 34s | n/a | 0 | n/a | n/a | Handoff 2/4 |
| A: Baseline | Y | 37s | 12s | 0 | Yes (`diarize=true`) | `/v1/listen` `summarize=v2` | 4/5 |
| B: llms.txt | Y | 70s | 29s | 0 | No | `/v1/read` (after misreading the response) | 5/5 |
| C1: Docs MCP, server down | Y | 75s | 49s | 0 | No | `/v1/read` (summarization page not found) | 5/5 |
| C-alt: Other docs MCP | Y | 52s | 39s | 0 | Yes (`diarize=true`) | `/v1/listen` `summarize=v2` | 4/5 |

Full data: [`experiments/agent-dx/results.md`](experiments/agent-dx/results.md). Run logs: [`experiments/agent-dx/runs/`](experiments/agent-dx/runs/).

## Hypotheses
| # | Hypothesis | Result | Evidence |
|---|---|---|---|
| 1 | Docs access reduces time to a working build and errors | **Rejected** | Docs runs were slower (70s, 75s, and 52s vs. 37s). Errors didn't drop: B misread a response and drew a false conclusion. Docs improved *currency* in B, but not in C-alt |
| 2 | Without docs, the agent picks up outdated patterns | **Supported, by a different mechanism** | A used the deprecated `diarize=true` from the model's prior knowledge, with no web search. C-alt used it too, following a Deepgram recipe returned by the docs MCP server |
| 3 | Without a key, the agent's handoff instructions are incomplete | **Supported, but reframed** | Handoff scored 2/4 (no signup or key-creation steps). More important: the agent didn't stop. It built untested code and deferred verification to the human |
| 4 | The agent uses a separate LLM for summarization | **Rejected** | Every run, including C-alt, used Deepgram's own summarization. But in A, the agent recommended switching to another LLM for better summaries |

## Key findings

### 1. Prior knowledge goes stale silently
In Condition A, the agent consulted no docs and finished in 37 seconds. It worked, but it used `diarize=true`, which Deepgram's Diarization page marks deprecated in favor of `diarize_model`. Nothing told the agent or the developer: in a later controlled test, requests using `diarize=true` returned no warnings. 

**Fast and working is not the same as current.** For a well-known API, the agent experience is set by the model's training data first, and docs only matter if the agent decides to look.

*Evidence:* the [Condition A run log](experiments/agent-dx/runs/condition-a.md) ("Discovery": no docs fetched; "Integration quality": the deprecated parameter, scored under item 3); Deepgram's [Diarization page](https://developers.deepgram.com/docs/diarization), which marks `diarize=true` deprecated; and the controlled test in the [Condition B run log](experiments/agent-dx/runs/condition-b.md) ("Where it went wrong"), where requests using `diarize=true` returned no warnings.

### 2. Docs made the agent current
With llms.txt (B), the agent read the Diarization page and used the current `diarize_model=latest` instead of the deprecated `diarize=true`.

B's speaker labels were also clearly better than A's on the same clip, but the newer setting doesn't appear to be the reason. C1 used the same setting as B and produced speaker turns identical to A's. The difference most likely comes from how B's code built speaker turns: from individual words, merging one-word speaker flips, rather than from Deepgram's utterance segments. So the agent's own integration choices changed what the developer saw. *(One clip; a controlled test to separate the two effects is pending.)*

**Cost:** roughly double the time (70s vs. 37s; 29s vs. 12s to first call) for four docs fetches.

Not every docs path had this effect: in C-alt, the docs server led the agent to the deprecated parameter (finding 10).

*Evidence:* the [Condition B run log](experiments/agent-dx/runs/condition-b.md) ("Discovery": the pages fetched and the diarizer chosen; "Timeline": each step with timestamps); the side-by-side transcripts in [`results.md`](experiments/agent-dx/results.md) ("Output comparison: A vs. B"); and the timings in the results table above.

### 3. A misread response became a false claim about the product
In B, the agent checked for the summary at the top level of the response. Deepgram returns it under `results.summary`, and the response metadata even showed summarization had run. The agent concluded summarization doesn't work with nova-3, built a workaround through `/v1/read`, and **wrote the false claim into the README.**

A controlled test confirmed the API was fine: four requests varying the diarizer and `language` setting, including B's exact settings, all returned a summary. 

The page itself is correct: its response example shows `summary` inside `results`. But the agent read a condensed version of the page that kept only "same level as `channels`" and dropped the example. Docs are increasingly read through summaries like this, so exact paths need to be in the prose, not only in examples. 

**When agents misread docs, developers inherit the mistake as a statement about the product.**

*Evidence:* the [Condition B run log](experiments/agent-dx/runs/condition-b.md) ("Timeline": the agent checks the top level of the response; "Where it went wrong": the four-request controlled test); Deepgram's [Summarization page](https://developers.deepgram.com/docs/summarization), whose response example shows `summary` inside `results`; and the Condition B entry in the [friction log](FRICTION_LOG.md).

### 4. Loud errors work; agents recover
In C1, the docs MCP server's search tool returned an explicit error ("Failed to fetch from FAI chat service"). The agent recognized the outage within a second and switched to fetching pages directly. 

A clear error was handled well; the cost of the outage showed up later, in what the agent couldn't find (finding 5).

*Evidence:* the [Condition C1 run log](experiments/agent-dx/runs/condition-c1.md) ("Timeline": both searches failed at 12:17:27 UTC with the error above, and the agent was fetching pages directly five seconds later).

### 5. Agent-facing infrastructure is now production infrastructure
With no hint in the prompt, the C1 agent went to the docs MCP server first. Its search failed at every check from 8:17 AM ET on October 3 to 12:50 PM ET on October 4, more than 28 hours. 

The outage was hard to see from the outside: the server's address responded normally, and Claude Code reported it as connected. Only an actual search showed that its search backend was failing. The server is provided by Deepgram's docs platform, so this part of the agent experience depends on a vendor's service.

Without search, the agent fetched three pages by guessing their addresses: the Models overview, Text Intelligence (twice), and Diarization. It never reached the Summarization page, so the only summarization it saw was the text version on the Text Intelligence page. Its final message told the developer that "the Deepgram docs I could reach describe summarization only on the text endpoint, /v1/read," and the README it wrote repeats the claim. That's false: in Condition A, audio summarization (`summarize=v2` on `/v1/listen`) returned a summary. Both B and C1 ended up with an inaccurate claim about Deepgram in their README, for different reasons.

*Evidence:* the [Condition C1 run log](experiments/agent-dx/runs/condition-c1.md) ("Timeline" and "Discovery": the pages fetched; "Where it went wrong": the agent's final message; "Server health checks": every check from October 3 to 4; "Where the failure is": the server responding while its search failed); and the [Condition A run log](experiments/agent-dx/runs/condition-a.md) for the working audio summary.

### 6. Agents defer when they can't verify
Without a key (Condition 0), the agent didn't stop to ask. It built the full tool in 34 seconds, checked for the key only at the very end, and handed verification to the human. It was honest about what it hadn't tested and never faked a result. But nothing let it confirm its integration before a human signed up and created a key.

*Evidence:* the [Condition 0 run log](experiments/agent-dx/runs/condition-0.md) ("What the agent did": the key check was its final command; "Its instructions to the human" and "Handoff quality": the 2/4 score).

### 7. Feature-by-model compatibility isn't stated anywhere an agent looks
In Condition 0, the agent couldn't confirm whether nova-3 supports summarization: the Summarization page says "Nova," which the Models page labels as legacy, and the SDK's type signature lists models and features separately. 

In B, the agent asked the Models page the same question and got no answer. Code examples also disagree by language (JavaScript and Java use `nova-3`; Python and Go use `nova-2`).

*Evidence:* the [Condition 0 run log](experiments/agent-dx/runs/condition-0.md) ("Notes for the memo"); the [Condition B run log](experiments/agent-dx/runs/condition-b.md) ("Where it went wrong"); and three entries in the [friction log](FRICTION_LOG.md) on the "Nova" wording, the SDK type signature, and the code examples on the [Audio Intelligence page](https://developers.deepgram.com/docs/audio-intelligence).

### 8. Agents evaluate products, not just integrate them
In A, the agent called Deepgram's summarizer "fairly literal" and suggested sending the transcript to another LLM instead. 

Agents act as reviewers and advisors to developers, and can steer them away from a feature.

*Evidence:* the agent's final message, recorded in the [Condition A run log](experiments/agent-dx/runs/condition-a.md) ("Notes for the memo").

### 9. Agents and people are sent to different docs servers
Deepgram advertises two docs MCP servers from two providers, plus a third pointer. The Markdown and `llms.txt` versions of the docs point agents to `_mcp/server`. The Agentic developer tools page lists only `kapa/mcp`. A note hidden in the HTML pages points agents to `llms.txt`. The Home page shows it most clearly: its HTML version links people to the kapa server, while its Markdown version tells agents to use `_mcp/server`.

The two servers also differ in setup. `_mcp/server` needs no authentication; `kapa/mcp` requires a browser sign-in or an API key header, and the setup page mentions neither. A public pull request in Deepgram's skills repository ([#11](https://github.com/deepgram/skills/pull/11), verified on 2026-09-18 and since merged) found this about two weeks before this study and switched the skills' instructions to `_mcp/server`, but the public setup page still lists only `kapa/mcp`.

*Evidence:* the [friction log](FRICTION_LOG.md) ("Examples: two docs MCP servers": links to each page, both servers' setup commands, and pull request #11 in detail); Deepgram's [Agentic developer tools page](https://developers.deepgram.com/developer-tools/agentic-tools); and the sign-in details in the [Condition C-alt run log](experiments/agent-dx/runs/condition-c-alt.md) ("Setup check").

### 10. What a docs search returns matters as much as whether it works
In C-alt, the docs MCP server worked, and the agent used it first. Its two searches returned 28 results, mostly code recipes, examples, and community discussions. For summarization, the official pages appeared near the top, and the agent correctly used audio summarization on `/v1/listen`, which B and C1 missed. For diarization, the top result was Deepgram's own recipe using the deprecated `diarize=true`, and the Diarization page explaining its replacement never appeared. The agent used the deprecated parameter. Its output matched Condition A's exactly, but so did C1's, which used the current setting, so on this clip the deprecated parameter didn't visibly change the speaker labels.

**Docs only make agents current if the content they surface is current.** Recipes are valuable to agents, but they fall behind unless they're updated along with the docs.

*Evidence:* the [Condition C-alt run log](experiments/agent-dx/runs/condition-c-alt.md) ("Search results": the top results for both searches, including the recipe's code; "Integration quality": the deprecated parameter); and the output comparisons in [`results.md`](experiments/agent-dx/results.md).

## What I'd do next
Give agents one dependable docs MCP server: pick one canonical server and point every page, note, and setup instruction to it, with its authentication documented; point agents to `llms.txt` when search fails; monitor it with real searches instead of connection checks; keep basic search working when the AI backend is down; and keep recipes current when parameters are deprecated. The full plan, plus four other opportunities ranked, is in [`PROPOSAL.md`](PROPOSAL.md).

## Limitations
- **One run per condition**, one agent (Claude Code), one model (Claude Sonnet 5.5), one task, one 2-minute clip. These are signals, not statistics.

- **Condition C wasn't administered.** The search on the docs MCP server advertised in the agent-facing docs was down throughout the study. C-alt tested a different server, from a different provider, so it doesn't stand in for Condition C.

- **C-alt was added after the other results were known.** It was planned and committed (Amendment 2) before the run, but the decision to test a second server came from what C1 revealed. Its setup also needed a one-time sign-in and two connection retries before the run started. That was setup, not counted as intervention, but it's part of the experience.

- **Setup differences, documented in the run logs:** Condition 0 ran in the Claude desktop app with the SDK preinstalled; A-C ran in the CLI with the SDK uninstalled. `requests` and `httpx` were installed for A-C, which may have shaped the agents' choice of raw HTTP.

- **My first read of B was wrong.** I initially logged B as a silent API failure, based on the agent's own account. The controlled test showed the agent misread the response. Agent claims need independent checks.

- **Agents read docs through a summarizing fetch tool**, so what they "read" was a condensed version of each page.

- **Runs happened on different days**, so a docs or API change between runs can't be fully ruled out.

- **A correction to Amendment 2.** Amendment 2 in `PROTOCOL.md` describes `_mcp/server` as advertised "on every Deepgram docs page." More precisely, it's advertised in the Markdown and `llms.txt` versions of the pages; the HTML pages carry a hidden note pointing agents to `llms.txt` instead (see the friction log).
