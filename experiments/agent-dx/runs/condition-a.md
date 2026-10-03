# Run: Condition A (Baseline)  |  Date: 2026-10-02 (US Eastern)

**Agent / model:** Claude Code CLI, Claude Sonnet 5.5
**Start:** 00:34:53 UTC (8:34:53 PM ET)    **First successful call:** 00:35:06 UTC    **End:** 00:35:30 UTC
**Wall time:** 37 seconds    **Time to first successful call:** 12 seconds
**Source:** Claude Code session log (timestamps in UTC)

## Setup check (Amendment 1)
- [x] New, empty folder with a neutral name (`~/Code/dx-runs/run-a`)
- [x] Test clip in folder: `meeting.wav` (same clip for A-C)
- [x] `deepgram-sdk` uninstalled (log confirms: the agent's package check found no Deepgram package)
- [x] `DEEPGRAM_API_KEY` exported in this terminal only (log confirms: agent's check returned "yes")
- [x] Prompt includes the amendment sentence about `./meeting.wav`
- [x] Condition-specific setup: none for A

**Environment note:** `requests` and `httpx` were installed globally. The agent checked for them,
and that likely shaped its choice of raw HTTP over the SDK. Keep the environment identical for B and C.

## Outcome
- Working end to end (transcribe + speaker labels + meeting summary)? **Y** (confirmed by my own run with the key set)
- Human interventions: 0
- Errors hit and retries: 0

## Timeline
| Time (UTC) | Elapsed | Event |
|---|---|---|
| 00:34:53 | 0s | Prompt submitted |
| 00:34:56 | 3s | Checked folder, Python version, installed packages, and that the key was set |
| 00:35:02 | 9s | Called Deepgram directly with `curl` (nova-3, diarize, utterances, smart_format, summarize=v2) |
| 00:35:06 | 12s | **First successful call:** HTTP 200 with transcript, utterances, and summary |
| 00:35:16 | 23s | Wrote `meeting_summary.py`, `requirements.txt`, `README.md`, then ran the tool |
| 00:35:25 | 32s | Tool's own run succeeded: 2 speakers, summary, transcript |
| 00:35:30 | 37s | Final report |

## Discovery
- First Deepgram source consulted: **none.** No web search, no docs pages fetched.
- Doc pages / URLs fetched: none.
- SDK or raw HTTP? **Raw HTTP** (`requests`), after confirming no Deepgram SDK was installed.
- Models / endpoints chosen: `nova-3` on `/v1/listen`. Current.
- Summarization: **Deepgram's own** (`summarize=v2`).

## Integration quality
| # | Check | Pass? | Evidence |
|---|---|---|---|
| 1 | Current STT model | Yes | `nova-3`, listed as current on https://developers.deepgram.com/docs/models |
| 2 | Diarization on and working | Yes | `diarize=true`; output separated 2 speakers. Some short replies attributed to the wrong speaker (agent noted this) |
| 3 | Current SDK/API patterns | Yes | `/v1/listen` with `Authorization: Token` header, current REST pattern |
| 4 | Uses Deepgram's own features | Yes | `summarize=v2`, no separate LLM |
| 5 | Clear API key error handling | Yes | Confirmed by my own run with the key unset: printed "DEEPGRAM_API_KEY is not set." |
| | **Score** | **5/5** | |

## Where it went wrong
Nothing in the integration. Quality issues were in the output, not the code: "Sassafras" was transcribed
as "Sashfak," which carried into the summary, and a few short replies were assigned to the wrong speaker.
Summary prose quality is out of scope for scoring (see protocol).

## Notes for the memo
- **The docs were never consulted.** A frontier model with a recent knowledge cutoff completed the task in
  37 seconds from prior knowledge alone. For a common use case of a well-known API, the agent experience is
  set by what the model already knows, not by the docs. This creates a ceiling effect for B and C.
- **It verified before building.** It tested the API with `curl` first, then wrote the tool. The opposite of
  Condition 0's defer behavior, because a key was available.
- **Answers the Condition 0 compatibility question:** `nova-3` with `summarize=v2` returned a successful summary.
  The Summarization page's "Nova" wording is a docs clarity issue, not a product limitation.
- **The agent evaluated the product for the developer.** It called Deepgram's summarizer "fairly literal" and
  suggested sending the transcript to Claude instead. Agents don't just integrate; they recommend, and can steer
  developers away from a feature.
- **No SDK.** With the SDK absent and `requests` present, it went straight to the REST API. The SDK was neither
  needed nor discovered.
