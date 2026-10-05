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
- Errors hit and retries: 0 HTTP errors. 1 **misread response**: the agent looked for the summary at the top level of the JSON response instead of under `results`, concluded none was returned, and switched to a second endpoint. (Originally logged as a silent failure; corrected after the controlled test below.)

## Timeline
| Time (UTC) | Elapsed | Event |
|---|---|---|
| 03:19:29 | 0s | Prompt submitted |
| 03:19:32 | 3s | Checked folder, Python version, and that the key was set |
| 03:19:35 | 6s | Fetched `llms.txt` |
| 03:19:43 | 14s | Fetched three docs pages in parallel (`.md` versions): Summarization, Models overview, Diarization |
| 03:19:54 | 25s | Called `/v1/listen` with `curl` (nova-3, `diarize_model=latest`, utterances, smart_format, `summarize=v2`) |
| 03:19:58 | 29s | **First successful call:** HTTP 200 with transcript and utterances. The agent checked `d.get('summary')` at the top level, got `None`, and missed `results.summary` |
| 03:20:02 | 33s | Tried Deepgram's Text Intelligence endpoint, `/v1/read?summarize=true`, on the transcript. Summary returned |
| 03:20:22 | 53s | Wrote `summarize_meeting.py` (listen, then read) and ran it |
| 03:20:26 | 58s | Tool's own run succeeded: 2 speakers, summary, transcript |
| 03:20:34 | 65s | Wrote `README.md` |
| 03:20:39 | 70s | Final report |

## Discovery
- First Deepgram source consulted: `https://developers.deepgram.com/llms.txt`

- Doc pages fetched (in order): llms.txt; then `/docs/summarization.md`, `/docs/models-languages-overview.md`, `/docs/diarization.md`

- SDK or raw HTTP? **Raw HTTP**, Python standard library only (`urllib`). No dependencies.

- Models / endpoints chosen: `nova-3` on `/v1/listen`; `/v1/read` for summarization. Diarization via `diarize_model=latest` (the current setting), learned from the Diarization page. The response included a `diarize_info` field showing which diarizer ran, but its value wasn't captured, so that isn't confirmed.

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
- **A misread response, then a wrong diagnosis.** The agent's check looked for `summary` at the top level of the response. Deepgram returns it under `results.summary`. The response's metadata even included `summary_info`, showing summarization had run. The agent concluded that `summarize` doesn't work with nova-3, and wrote that into the README.

- **Controlled test (2026-10-03):** four `curl` calls varying the diarizer (`diarize=true` vs.
  `diarize_model=latest`) and `language=en`. **All four returned a summary under `results.summary`, with no warnings,** including Condition B's own settings. Summarization works with nova-3 and `diarize_model=latest`; the API behaved correctly. The failure was in how the agent read the response.

- **Likely contributor:** the Summarization page's response example is correct: it shows `summary` inside `results`, next to `channels`. But the agent read the page through a summarizing fetch tool, which reduced the example to "at the same level as `channels`" and dropped the full structure. The agent then looked at the top level.
  Verified against https://developers.deepgram.com/docs/summarization on 2026-10-03.

- **Misinformation passed to the developer.** The README the agent wrote states: "Passing `summarize` directly to `/listen` returned no summary with nova-3 in testing, so the tool summarizes the transcript text instead." That's false: the summary was in the response, under `results.summary`. A developer reading it would conclude that Deepgram's audio
  summarization doesn't work with nova-3, and the tool makes a second API call (`/v1/read`) it doesn't need.

- **Feature support isn't stated on the Models page.** The agent fetched the Markdown version of the Models overview
  (https://developers.deepgram.com/docs/models-languages-overview) and asked whether nova-3 supports diarization, summarization, and smart formatting. The answer it got back: the page doesn't specify. For diarization, the answer does exist elsewhere: the Diarization page (https://developers.deepgram.com/docs/diarization), which the agent also fetched, lists all Nova batch models, including Nova-3, as compatible. For summarization, it stayed unclear: the Summarization page says only "Nova." That summarization gap is the same one the agent hit in Condition 0.

## Notes for the memo
- **Docs made the agent more current.** (Diarization page verified: `diarize=true` is deprecated.) Only the docs-guided agent used the current diarization setting. The baseline used the deprecated `diarize=true` from prior knowledge. B's better speaker labels most likely came from how its code built speaker turns, not from the setting: see the output comparisons in `results.md`.

- **Docs cost time.** 70s vs. 37s, and 29s vs. 12s to first call. Four fetches added roughly 20 seconds.

- **The `.md` versions worked as intended.** llms.txt pointed the agent to `.md` pages, and it fetched those.

- **A parsing slip became a false claim about the product.** The API worked; the agent misread where the summary was, and the developer received a README saying a Deepgram feature doesn't work.

- **Relative descriptions don't survive agents.** "Same level as `channels`" is clear to a human looking at the example, but after a summarizing fetch, an exact path like `results.summary` is what an agent needs.
