# Observations: Deepgram docs and APIs, as used by an AI coding agent

What was observed between September 30 and October 5, 2026, with no interpretation of causes. Analysis and recommendations are in [`FINDINGS.md`](FINDINGS.md) and [`PROPOSAL.md`](PROPOSAL.md); the raw records are in [`experiments/agent-dx/runs/`](experiments/agent-dx/runs/).

All times are US Eastern unless marked UTC.

## 1. Docs MCP server at `developers.deepgram.com/_mcp/server`

- **Search failed at every check.** The `searchDocs` tool returned `Search failed: {"error":"Failed to fetch from FAI chat service"}` at each of these times:

  | Date | Time | Context |
  | --- | --- | --- |
  | Oct 3 | 8:17 AM | Two searches during Condition C1 |
  | Oct 3 | 8:45 AM | Health check 1 |
  | Oct 3 | 12:33 PM | Health check 2 |
  | Oct 4 | 12:50 PM | Health check 3 |

- **The server itself responded.** Opening the address in a browser on Oct 3 returned a server description: `fern-docs-mcp-server`, version 1.0.0, one tool (`searchDocs`). In every check above, the `searchDocs` tool could be called, and returned the error shown. When checked with `claude mcp list`, the server was listed as connected.
- **Where it is advertised.** The Markdown versions of docs pages and the `llms.txt` indexes checked open with a note to connect AI clients to this address. Checked through search-index snapshots on Oct 3: https://developers.deepgram.com/home.md, https://developers.deepgram.com/home/llms.txt, https://developers.deepgram.com/docs/stt/getting-started/llms.txt, and https://developers.deepgram.com/docs/flux-tts/overview.md.

## 2. Docs MCP server at `api.dx.deepgram.com/kapa/mcp`

- **Where it is advertised.** The Agentic developer tools page (https://developers.deepgram.com/developer-tools/agentic-tools) lists this address, with `https://deepgram.mcp.kapa.ai` as an alternative, and gives a one-line Claude Code setup command. On Oct 3, the page's Docs MCP server section did not mention `_mcp/server` or any sign-in requirement.
- **Authentication.** After adding the server, `claude mcp list` showed `! Needs authentication`. The sign-in page offered sign-in through an email provider or an email address. Sign-in with a Google account succeeded after granting permission to share basic account information; it took about 22 seconds.
- **Connection (Oct 3, UTC).** First connection: shown as connected with no tools listed. 19:02:52: `Version negotiation probe timed out after 5000ms`. 19:05:41: connected, with the tool `search_deepgram_knowledge_sources`.
- **Search results.** In Condition C-alt, a diarization query returned 13 results, of which 1 was from developers.deepgram.com. The top result was the Python diarization recipe in the `deepgram/recipes` repository, which uses `diarize=True`. The Diarization docs page was not among the results.
- **Public pull request on the same servers.** Pull request #11 in the public `deepgram/skills` repository (https://github.com/deepgram/skills/pull/11, verified by its author on 2026-09-18, merged per pull request #16) reports HTTP 401 for unauthenticated connections to both kapa addresses, and HTTP 200 for `_mcp/server`. A review comment on the same pull request reports HTTP 200 from `api.dx.deepgram.com/kapa/mcp` when a Deepgram API key is sent as an `Authorization: Token` header.

## 3. Other agent-facing pointers

- **HTML docs pages:** the Summarization page (https://developers.deepgram.com/docs/summarization) contains a note for AI agents, hidden from view but present in the page source, pointing to `llms.txt` and the `.md` page versions (verified in the page source on Oct 4). Search-index snapshots of other HTML pages on Oct 3 show the same note.
- **Home page,** per search-index snapshots on Oct 3: the HTML version's "MCP Server" card links to the Agentic developer tools page; the Markdown version (https://developers.deepgram.com/home.md) points to `_mcp/server`.

## 4. API behavior (controlled test, Oct 3)

Four requests to `/v1/listen` with `model=nova-3` and `summarize=v2`, on the same 2:14 test clip:

| Diarization setting | `language=en` | `results.summary` returned | `metadata.warnings` |
| --- | --- | --- | --- |
| `diarize=true` | Yes | Yes | None |
| `diarize_model=latest` | No | Yes | None |
| `diarize_model=latest` | Yes | Yes | None |
| `diarize=true` | No | Yes | None |

The Diarization docs page (https://developers.deepgram.com/docs/diarization) describes `diarize=true` as deprecated and recommends `diarize_model`.

## 5. Docs content

- **Summarization page** (https://developers.deepgram.com/docs/summarization): lists availability as "Nova." The Models page labels "Nova" as a legacy model (Nova 1). The page's response example shows `summary` inside `results`.
- **Models overview** (https://developers.deepgram.com/docs/models-languages-overview): when the agent in Condition B asked whether nova-3 supports diarization, summarization, and smart formatting, the content returned to it said the page does not specify.
- **Audio Intelligence page** (https://developers.deepgram.com/docs/audio-intelligence): the JavaScript and Java code examples use `nova-3`; the Python and Go examples use `nova-2`.
- **Voice agent demo** (https://github.com/deepgram-devs/deepgram-voice-agent-demo, listed as the "Basic demo" on https://developers.deepgram.com/docs/build-a-voice-agent): the README and the repository's About section link to https://deepgram.com/agent, which returned a 404 on Sep 30 and Oct 3.

## 6. What the agent did in each run

Claude Code with Claude Sonnet 5.5, the same task prompt naming Deepgram, and the same 2:14 test clip. One run per condition, except Condition A, which was repeated (section 8).

| Run | Docs consulted | Diarization setting | Summarization | Wall time | Human interventions |
| --- | --- | --- | --- | --- | --- |
| 0: no API key | Summarization page | `diarize=true` (in code; not run) | `summarize=v2` on `/v1/listen` (in code; not run) | 34s | 0 |
| A: no docs pointer | None | `diarize=true` | `summarize=v2` on `/v1/listen` | 37s | 0 |
| B: `llms.txt` pointer | `llms.txt`, Summarization, Models overview, Diarization | `diarize_model=latest` | `/v1/read` | 70s | 0 |
| C1: `_mcp/server` connected | Search failed; then Models overview, Text Intelligence (twice), Diarization | `diarize_model=latest` | `/v1/read` | 75s | 0 |
| C-alt: `kapa/mcp` connected | Two searches | `diarize=true` | `summarize=v2` on `/v1/listen` | 52s | 0 |

Specific observations:

- **Condition 0:** the agent built the tool, then checked for the API key as its final command. Its README said to set `DEEPGRAM_API_KEY`; it did not mention signing up or creating a key.
- **Condition B:** the agent's check read `summary` from the top level of the response and got `None`; the response contained `results.summary`. The README it wrote states: "Passing `summarize` directly to `/listen` returned no summary with nova-3 in testing, so the tool summarizes the transcript text instead."
- **Condition C1:** the agent's final message stated that "the Deepgram docs I could reach describe summarization only on the text endpoint, /v1/read."
- **Condition A:** the agent's final message described Deepgram's summarizer as "fairly literal" and suggested sending the transcript to another model for better summaries.

## 7. Output comparison (same clip, same model)

- Runs A, C1, and C-alt produced identical speaker turns, line for line. A and C-alt used `diarize=true`; C1 used `diarize_model=latest`. All three built speaker turns from Deepgram's utterance segments.
- Run B, which used `diarize_model=latest` and built speaker turns from word-level speaker labels, assigned several replies to different speakers than runs A, C1, and C-alt did.

## 8. Repeat runs of Condition A (Oct 5)

Four more runs with Condition A's setup (A2 to A5), for five in total:

- **No docs consulted:** 5 of 5.
- **Diarization setting:** `diarize=true` in 5 of 5.
- **Summarization:** `summarize=v2` on `/v1/listen` in 5 of 5.
- **Wall time:** 37, 55, 32, 26, and 34 seconds. **Time to first successful call:** 12, 16, 11, 16, and 12 seconds.
- **In their final messages,** 4 of 5 agents said some replies were assigned to the wrong speaker; 4 of 5 described a limitation of the summary (for example, "fairly literal," "a little clumsy"); 3 of 5 suggested or offered replacing the summary with another LLM.

## 9. Selection probe (Oct 5)

Five runs of the same task with no provider named, and no API keys set ("Use whichever speech-to-text service you think is best"):

- **Speech-to-text and speaker labels:** AssemblyAI in 5 of 5.
- **Summary:** Claude in 5 of 5.
- **Deepgram:** chosen in 0 of 5. In all five runs, the agent's first command searched the environment for variable names matching a list that included `DEEPGRAM` (for example, S1: `env | grep -iE 'ASSEMBLY|DEEPGRAM|OPENAI|ANTHROPIC'`). No Deepgram key was set, and no run used Deepgram.
- **Docs consulted:** none in 5 of 5.
- **Reason given for AssemblyAI,** in all five runs: transcription and speaker labels in one call or upload (for example, S1: "AssemblyAI does both in one call, so there's no separate diarization step").

## Not tested

- More than one run per condition, except Condition A (five runs) and the selection probe (five runs).
- Agents other than Claude Code, and models other than Claude Sonnet 5.5.
- Tasks other than pre-recorded transcription with speaker labels and a summary.
- Which provider other agents or models pick when the task doesn't name one.
- Whether `_mcp/server` was available between the checks listed above.
