# Proposal: Give agents one dependable docs MCP server

**Status:** For discussion · **Author:** Rami Moussawi · **Evidence:** [findings](FINDINGS.md), [Condition C1](experiments/agent-dx/runs/condition-c1.md), [Condition C-alt](experiments/agent-dx/runs/condition-c-alt.md), [friction log](FRICTION_LOG.md)

## Problem
Agents now look for Deepgram's docs through an MCP server, and the path they find is fragmented and fragile.

- **The server agents find was down for more than a day, with no visible status notice.** Starting at 8:17 AM ET on October 3, 2026, the search on `_mcp/server` failed at every check for more than 28 hours (the last at 12:50 PM ET on October 4), while the server still appeared connected. Without it, the agent guessed pages and told the developer that Deepgram documents summarization only for text, which is false.

- **Two servers, three pointers.** The Markdown and `llms.txt` versions of the docs point agents to `developers.deepgram.com/_mcp/server`. The Agentic developer tools page points people to `api.dx.deepgram.com/kapa/mcp`, from a different provider. A hidden note on the HTML pages points agents to `llms.txt` instead. A public pull request in Deepgram's skills repository ([#11](https://github.com/deepgram/skills/pull/11), verified on 2026-09-18 and since merged) switched the skills' setup instructions to `developers.deepgram.com/_mcp/server` about two weeks before this study, but the public setup page still lists only `api.dx.deepgram.com/kapa/mcp`.

- **The recommended server needs undocumented authentication, and its results can be out of date.** `kapa/mcp` requires a browser sign-in or an API key header, and its [setup page](https://developers.deepgram.com/developer-tools/agentic-tools) mentions neither. Once connected, its diarization search returned Deepgram's own recipe using the deprecated `diarize=true`, and the agent used it. The Diarization page explaining the replacement never appeared.

Deepgram's API worked throughout, so none of this would show up in API metrics.

## Target user
**Primary: the AI coding agent** integrating Deepgram for a developer. It needs a docs search that works, and when search fails, an error that tells it where else to look. Without that, it falls back to guessing pages and filling gaps from memory.

**Secondary: the developer supervising the agent.** They need the agent's claims about Deepgram to be accurate. They can't tell when an agent's statement came from incomplete docs access, so they take it as fact.

**Internal: Deepgram's docs and developer-experience team.** They need to know when agent-facing services stop working. Today, an outage like this one is easy to miss from the outside: the API is healthy, the server responds, and nothing reports an error.

## Proposed solution
**1. Pick one canonical docs MCP server, and point everything to it.** Update the Agentic developer tools page, the notes in the Markdown and `llms.txt` versions, the hidden note on the HTML pages, and Deepgram's skills and example READMEs so they all name the same server. State on the setup page whether it requires authentication, and how to provide it. If both servers stay, explain when to use each.

**2. Point agents to a fallback when search fails.** When the search tool can't return results, its error message should say the outage is temporary and direct the agent to the docs index at `https://developers.deepgram.com/llms.txt`. In Condition B, `llms.txt` led the agent straight to the right pages. This is a small change to an error message.

**3. Monitor agent-facing services with real queries.** Run a known search against the canonical server every few minutes and alert when it fails. Connection checks aren't enough: during the outage, the server responded and Claude Code reported it as connected, while every search failed.

**4. Keep search working when the AI backend is down.** If the AI search service fails or doesn't respond within a few seconds, the search tool falls back to a keyword search over the same docs and labels the results as limited. The quickest version matches the query against the page titles and descriptions already in `llms.txt` and returns the matching `.md` pages. In C1, this would have returned the Summarization page the agent never found. A fuller version would search the full page text, using an index built each time the docs are published.

**5. Keep search results current.** When a parameter is deprecated, update the recipes and examples that use it. For questions about a feature, return its docs page alongside code recipes, so the agent sees the current guidance. In Condition C-alt, the top diarization result was a recipe using the deprecated parameter, and the Diarization page never appeared.

**Order:** Part 1 comes first, since the other parts depend on which server is chosen. Parts 2 to 4 can then proceed in parallel, and part 5 is ongoing.

## Requirements
The last column links each requirement to the numbered part of the proposed solution above.

| Priority | Requirement | Solution part |
|---|---|---|
| Must | Every place that mentions the docs MCP server names the same one: the setup page, the Markdown and `llms.txt` notes, the hidden note on HTML pages, the skills, and example READMEs | 1 |
| Must | The setup page says whether the server needs authentication and how to provide it | 1 |
| Must | If search fails, the error message tells the agent the outage is temporary and where to find the docs index (`llms.txt`) | 2 |
| Must | A test search runs every 5 minutes, and the docs team gets an alert after 2 failures in a row | 3 |
| Must | The docs MCP server, `llms.txt`, and the `.md` page versions are tracked on Deepgram's status page or internal dashboard, like the API | 3 |
| Should | If the AI search is down, search still returns basic keyword results instead of an error | 4 |
| Should | Those basic results are labeled as limited, so the agent can tell the developer | 4 |
| Should | When a parameter is deprecated, the recipes and examples that use it are updated | 5 |
| Should | Questions about a feature return its docs page alongside code recipes | 5 |

The 5-minute interval and 2-failure threshold follow from the 10-minute detection target under "Success metric." They're starting points: if Deepgram sets a looser target for agent-facing docs, checks can run less often. Even hourly checks would have caught this outage.

## Non-goals
This proposal intentionally doesn't cover the items below. That doesn't mean they shouldn't be done: some are worth pursuing as separate efforts.

- **Overhauling search relevance.** Part 5 asks only that questions about a feature return its docs page and that deprecated recipes get updated. Broader ranking improvements are a separate effort.

- **Rewriting docs pages.** Part 5 updates recipes and examples when parameters are deprecated. Other docs changes, such as stating exact response paths in prose and making feature-by-model compatibility explicit, are listed under "Other opportunities."

- **Replacing the docs platform.** Part 1 asks Deepgram to choose which docs server is canonical, not to move its documentation to a different platform.

- **Changing the Deepgram API.** The API worked throughout. Nothing here touches it.

## Success metric
**Primary: time to detect an agent-facing docs outage.** Target: under 10 minutes. This outage persisted across every check for more than 28 hours, with no visible fix or notice. The docs team should know within minutes, as they would for an API outage.

**Supporting: search success rate.** The share of test searches that return results, tracked over time. Target: 99.9% per month, the same standard a team would expect of a production API.

**Supporting: pointers to a non-canonical server.** The number of places (the setup page, the Markdown and `llms.txt` notes, the hidden note on HTML pages, the skills, and example READMEs) that name a docs MCP server other than the canonical one. Target: zero, checked automatically.

**Guardrail: false alerts.** An alert is false when it fires but search is actually working, for example because of a brief network glitch at the monitoring location. Target: no more than one per week. If alerts fire too often for no reason, people start ignoring them, and a real outage gets missed.

**Validation:**
- **Fallback:** rerun this repo's Condition C while search is deliberately disabled. The fix works if the agent follows the error message to `llms.txt` and finds the Summarization page, instead of guessing.
- **Current results:** rerun Condition C-alt against the canonical server. The fix works if the agent uses `diarize_model` instead of the deprecated `diarize=true`.

## Risks and open questions
**Risks**

- **The fallback could fail too.** If the whole docs site goes down, the link to `llms.txt` won't help. It's still a strong fallback for this kind of outage, where only the AI search backend failed and the rest of the docs kept working.

- **Basic keyword results are weaker.** Agents might rely on lower-quality results without knowing it. Labeling them as limited (see requirements) reduces this risk.

- **Choosing one server means accepting its weaknesses until they're fixed.** `_mcp/server` needs no authentication, which suits unattended agents, but its search failed for hours. `kapa/mcp` requires authentication, which likely helps manage abuse and cost, and it searches recipes and discussions as well as docs. Neither is clearly better today.

- **Existing setups could break.** Developers and agents already configured with the server that isn't chosen need a migration path, such as keeping the old address working with a notice. Deepgram's skills repository already documents a naming conflict that users following older setup instructions run into.

- **Ownership is split.** The two servers come from two different providers. Deepgram can monitor them and get alerted, but a fix may depend on the provider. Who responds, and how quickly, needs to be agreed in advance.

**Open questions**

- **Is having two servers deliberate?** They may serve different purposes, for example one focused on docs and one that includes recipes and discussions. If so, the setup page should explain when to use each.

- **Can the docs platforms support this?** Customizing the error message and adding a keyword fallback depend on what each provider allows.

- **Who owns recipes and examples?** Updating them when a parameter is deprecated works best if it's part of the deprecation process itself.

- **Did Deepgram know about this outage, and how long did it last?** I observed failures at every check from 8:17 AM ET on October 3 to 12:50 PM ET on October 4.

- **How much agent traffic do the docs MCP servers get?** That determines how high this should rank against other work.

- **Do other agents behave the same way?** I tested one agent and one model. Tools like Cursor may use the servers differently.

## Other opportunities
Ranked by how strongly I'd pursue each next.

**1. A sandbox or test key, so agents can check their work before a human signs up.**

*Evidence:* without a key (Condition 0), the agent built the whole tool, couldn't test it, and left verification to the human. Its setup instructions covered only 2 of 4 onboarding steps.

*Why first:* it's the most strategic for growth, since onboarding is where developers convert. But it rests on a single run, and I'd want data on how often agents actually stall at this step.

*Looking ahead:* browser-capable agents may soon complete signups on their own, which makes a deliberate path for agents more important, not less: otherwise they'll work their way through flows built for people.

*Risks:* an open sandbox invites abuse, real customer data, and leaked keys. A verification-only sandbox that accepts only Deepgram's sample audio, with short-lived, rate-limited keys, would avoid most of them while still letting agents confirm their integration works.

**2. Deprecation warnings in API responses.**

*Evidence:* Conditions A and C-alt both used the deprecated `diarize=true`, and four later test calls using it returned no warnings. Neither the agents nor the developer were told.

*Why second:* the evidence is strong, and Deepgram already sends warnings in some cases. But it's an API change, which another team likely owns.

**3. Feature-by-model compatibility stated where agents look.**

*Evidence:* in Conditions 0 and B, agents couldn't confirm which models support summarization from the docs or the SDK. Code examples also use different models depending on the language.

*Why third:* it's a real gap, but it mostly affects how confident agents are, rather than whether they succeed.

**4. Exact response paths in the prose of feature pages.**

*Evidence:* in Condition B, the agent read a condensed version of the Summarization page, missed that the summary sits under `results`, and told the developer summarization doesn't work.

*Why fourth:* it's a small, cheap change with a clear payoff, but it's based on one run.
