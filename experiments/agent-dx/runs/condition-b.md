# Run: Condition B (llms.txt)  |  Date: 2026-10-02 (US Eastern)

**Agent / model:** Claude Code CLI, Claude Sonnet 5.5
**Start:** 03:19:29 UTC (11:19:29 PM ET)    **First successful call:** 03:19:58 UTC    **End:** 03:20:39 UTC
**Wall time:** 70 seconds    **Time to first successful call:** 29 seconds
**Source:** Claude Code session log (timestamps in UTC)

## Setup check (Amendment 1)
- [x] New, empty folder with a neutral name (`~/Code/dx-runs/run-b`)
- [x] Test clip in folder: `meeting.wav` (same clip as A)
- [x] `deepgram-sdk` uninstalled
- [x] `DEEPGRAM_API_KEY` exported in this terminal only (log confirms: agent's check returned "set")
- [x] Prompt includes the amendment sentence about `./meeting.wav`
- [x] Condition-specific setup: llms.txt line appended to the prompt

## Outcome
- Working end to end (transcribe + speaker labels + meeting summary)? **Y** (confirmed by my own run with the key set)
- Human interventions: 0
- Errors hit and retries: 0 HTTP errors. 1 **silent failure**: `summarize=v2` on `/v1/listen` returned
  HTTP 200 with no summary and no warning. The agent switched to a second endpoint.

## Timeline
| Time (UTC) | Elapsed | Event |
|---|---|---|
| 03:19:29 | 0s | Prompt submitted |
| 03:19:32 | 3s | Checked folder, Python version, and that the key was set |
| 03:19:35 | 6s | Fetched `llms.txt` |
| 03:19:43 | 14s | Fetched three docs pages in parallel (`.md` versions): Summarization, Models overview, Diarization |
| 03:19:54 | 25s | Called `/v1/listen` with `curl` (nova-3, `diarize_model=latest`, utterances, smart_format, `summarize=v2`) |
| 03:19:58 | 29s | **First successful call:** HTTP 200 with transcript and utterances, **but no summary** |
| 03:20:02 | 33s | Tried Deepgram's Text Intelligence endpoint, `/v1/read?summarize=true`, on the transcript. Summary returned |
| 03:20:22 | 53s | Wrote `summarize_meeting.py` (listen, then read) and ran it |
| 03:20:26 | 58s | Tool's own run succeeded: 2 speakers, summary, transcript |
| 03:20:34 | 65s | Wrote `README.md` |
| 03:20:39 | 70s | Final report |

## Discovery
- First Deepgram source consulted: `https://developers.deepgram.com/llms.txt`
- Doc pages fetched (in order): llms.txt; then `/docs/summarization.md`, `/docs/models-languages-overview.md`, `/docs/diarization.md`
- SDK or raw HTTP? **Raw HTTP**, Python standard library only (`urllib`). No dependencies.
- Models / endpoints chosen: `nova-3` on `/v1/listen`; `/v1/read` for summarization. Diarization via `diarize_model=latest` (newer diarizer; the response confirmed `arch: v2`), learned from the Diarization page.
- Summarization: **Deepgram**, but via Text Intelligence (`/v1/read`), not the `summarize` parameter on `/v1/listen`.

## Integration quality
| # | Check | Pass? | Evidence |
|---|---|---|---|
| 1 | Current STT model | Yes | `nova-3`; the Models overview names it the recommended general-purpose model |
| 2 | Diarization on and working | Yes | `diarize_model=latest`; output separated 2 speakers; agent also smoothed one-word speaker flips |
| 3 | Current SDK/API patterns | Yes | Used `diarize_model=latest`, which the Diarization page recommends over the deprecated `diarize=true` Verified on https://developers.deepgram.com/docs/diarization |
| 4 | Uses Deepgram's own features | Yes | Deepgram Text Intelligence (`/v1/read`) for the summary, no separate LLM |
| 5 | Clear API key error handling | Yes | Confirmed by my own run with the key unset: printed "DEEPGRAM_API_KEY is not set." |
| | **Score** | **5/5** | |

## Where it went wrong
- **Silent failure, then a wrong diagnosis.** `/v1/listen` with `summarize=v2` returned no summary and no warning.
  The agent concluded that `summarize` doesn't work with nova-3, and wrote that into the README. Condition A disproves this: nova-3 with `summarize=v2` returned a summary. Condition A's request differed in two ways:
  `diarize=true` instead of `diarize_model=latest`, and an explicit `language=en`. One of those is the likely cause.
  <!-- Fill in after the controlled test (after Condition C). -->
- **Misinformation passed to the developer.** The README now tells the developer something false about the product.
- **Feature-model compatibility still undocumented.** The agent asked the Models page whether nova-3 supports diarization and summarization; the page didn't say (same gap as Condition 0).

## Notes for the memo
- **Docs made the agent more current.** (Diarization page verified: `diarize=true` is deprecated.) Only the docs-guided agent used the newer diarizer. The baseline used the
  older `diarize=true` from prior knowledge.
- **Docs cost time.** 70s vs. 37s, and 29s vs. 12s to first call. Four fetches added roughly 20 seconds.
- **The `.md` versions worked as intended.** llms.txt pointed the agent to `.md` pages, and it fetched those.
- **Silent success appears to be a pattern, not a one-off.** The Diarization page itself describes `diarize=true` on some self-hosted deployments returning a successful response without speaker labels, calling this consistent with Deepgram's longstanding behavior when a requested model isn't present. (That case is self-hosted; this run used the hosted API.)
- **A silent failure is worse than an error.** An error would have told the agent what was wrong. A 200 with a missing field led it to a confident, false conclusion that it then passed on to the developer.
