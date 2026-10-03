# Run: Condition C, attempt 1 (Docs MCP)  |  Date: 2026-10-03 (US Eastern)

> **Status: condition not administered.** The Deepgram docs MCP server returned errors on both calls, so the agent fell back to fetching docs pages directly. This run measures **recovery from an agent-facing docs outage**, not the effect of the MCP server. See "Rerun" below.

**Agent / model:** Claude Code CLI, Claude Sonnet 5.5
**Start:** 12:17:20 UTC (8:17:20 AM ET)    **First successful call:** 12:18:09 UTC    **End:** 12:18:35 UTC
**Wall time:** 75 seconds    **Time to first successful call:** 49 seconds
**Source:** Claude Code session log (timestamps in UTC)

## Setup check (Amendment 1)
- [x] New, empty folder with a neutral name (`~/Code/dx-runs/run-c`)
- [x] Test clip in folder: `meeting.wav` (same clip as A and B)
- [x] `deepgram-sdk` uninstalled
- [x] `DEEPGRAM_API_KEY` exported in this terminal only (log confirms: agent's check returned "yes")
- [x] Prompt identical to A (no llms.txt line, no mention of MCP)
- [x] Condition-specific setup: docs MCP server added for this folder (`https://developers.deepgram.com/_mcp/server`); the agent found its `searchDocs` tool

## Outcome
- Working end to end (transcribe + speaker labels + meeting summary)? **Y** (confirmed by my own run with the key set)
- Human interventions: 0
- Errors hit and retries: **2 MCP errors** (`searchDocs` returned "Search failed: Failed to fetch from FAI chat service" twice). 0 Deepgram API errors. 1 output fix: the summarizer read "Speaker 1" as a dog's name, so the agent relabeled speakers as A/B in the text sent for summarizing, and re-ran.

## Timeline
| Time (UTC) | Elapsed | Event |
|---|---|---|
| 12:17:20 | 0s | Prompt submitted |
| 12:17:24 | 4s | Checked folder, Python, audio file, and key; loaded the docs MCP `searchDocs` tool on its own |
| 12:17:26 | 6s | Two `searchDocs` queries in parallel |
| 12:17:27 | 7s | **Both failed:** "Failed to fetch from FAI chat service." Agent: "The docs search is down" |
| 12:17:32 | 12s | Fell back to fetching three docs pages directly: Models overview, Text Intelligence, Diarization |
| 12:17:42 | 22s | Fetched Text Intelligence again for the `/v1/read` request format |
| 12:18:02 | 42s | Wrote `meeting_summary.py` (`/v1/listen`, then `/v1/read`) and ran it. No `curl` test first |
| 12:18:09 | 49s | **First successful call** (via its own tool). Summary called "Speaker 1" a dog |
| 12:18:14 | 54s | Relabeled speakers A/B in the summarizer input; re-ran. Summary improved |
| 12:18:29 | 69s | Wrote `README.md` |
| 12:18:35 | 75s | Final report |

## Discovery
- **Did the agent use the MCP server unprompted?** Yes. It loaded `searchDocs` as its very first docs step.
- First Deepgram source consulted: docs MCP `searchDocs` (failed), then `/docs/models-languages-overview`
- Doc pages fetched: Models overview, Text Intelligence (twice), Diarization. HTML pages, not `.md`; no `llms.txt`.
  **Never reached the Summarization page.**
- SDK or raw HTTP? Raw HTTP, Python standard library only.
- Models / endpoints chosen: `nova-3` on `/v1/listen` with `diarize_model=latest`; `/v1/read` for the summary.
- Summarization: Deepgram, via Text Intelligence (`/v1/read`). It concluded summarization exists only for text, because the Text Intelligence page it guessed covers only `/v1/read`.

## Integration quality
| # | Check | Pass? | Evidence |
|---|---|---|---|
| 1 | Current STT model | Yes | `nova-3`, named the recommended general-purpose model on the Models overview |
| 2 | Diarization on and working | Yes | `diarize_model=latest`; 2 speakers separated |
| 3 | Current SDK/API patterns | Yes | `diarize_model=latest`, per https://developers.deepgram.com/docs/diarization |
| 4 | Uses Deepgram's own features | Yes | Deepgram Text Intelligence (`/v1/read`), no separate LLM. It missed audio summarization on `/v1/listen` |
| 5 | Clear API key error handling | Yes | Confirmed by my own run with the key unset: printed a "key not set" error |
| | **Score** | **5/5** | |

## Where it went wrong
- **The docs MCP server was down.** Both calls returned an explicit error. Credit to the server: the error was clear, and the agent recovered immediately.
- **Without search, discovery degraded.** It guessed page URLs, never found the Summarization page, and told the developer that Deepgram documents summarization "only on the text endpoint." The README repeats this. This is false as:
  `summarize=v2` on `/v1/listen` exists and worked in Condition A.
- **Summarizer sensitive to input format.** Sending "Speaker 1:" lines to `/v1/read` produced a summary about a dog named Speaker 1. The agent caught it and worked around it.

## Rerun
The protocol's Condition C (MCP-assisted) was not administered because of a server-side failure, not because of the result. This run is kept as recorded evidence. A rerun was planned once the server recovered. It wasn't run because the server's search was still failing at the last health check (below). Instead, Amendment 2 added Condition C-alt, which tests Deepgram's other documented docs MCP server. See `runs/condition-c-alt.md`.

### Server health checks
Each check uses a separate scratch session (not a run folder), the same server URL, and a neutral query ("pricing") so the check can't influence the experiment.

| Date / time (ET) | Result | Error |
|---|---|---|
| 2026-10-03, 8:17 AM | Failed (during the run) | `Search failed: {"error":"Failed to fetch from FAI chat service"}` |
| 2026-10-03, 8:45 AM | Failed (health check 1; searchDocs call at 12:45:26 UTC) | `Search failed: {"error":"Failed to fetch from FAI chat service"}` |
| 2026-10-03, 12:33 PM | Failed (health check 2; searchDocs call at 16:33:43 UTC) | `Search failed: {"error":"Failed to fetch from FAI chat service"}` |

No check succeeded, so the rerun didn't go ahead.

### Where the failure is
Opening the server address in a browser (2026-10-03) returns a valid description of the server:
`fern-docs-mcp-server`, version 1.0.0, offering one read-only tool, `searchDocs`, with setup instructions that match the Claude Code command used in this run. So the server itself responds; **its search backend is what fails** ("Failed to fetch from FAI chat service").

- **It looks healthy from the outside.** The server description loads, and `claude mcp list` reported "connected" each time. Neither runs a search, so neither detects this outage. Only a real query does.
- **It's provided by Deepgram's docs platform.** The server name indicates Fern, the platform behind Deepgram's documentation site. "FAI" is likely Fern's AI search service (an inference from the name). Deepgram's agent docs experience depends on that service.
- **The setup was correct.** The server's own Claude Code instructions use the same address and transport as this run, so the configuration wasn't the cause.

## Notes for the memo
- **Agents will use an MCP server unprompted.** No hint in the prompt; it went to `searchDocs` first.
- **Agent-facing infrastructure is now production infrastructure.** When the docs MCP server failed, the agent's path to the docs got worse, and it passed a false claim to the developer.
- **Explicit errors work.** This failure was loud, and the agent adapted in seconds.
- **Two runs, two different reasons, same detour.** B (misread response) and C1 (discovery failure) both ended up summarizing via `/v1/read` instead of `/v1/listen`, and both wrote an inaccurate claim about Deepgram into the README.
