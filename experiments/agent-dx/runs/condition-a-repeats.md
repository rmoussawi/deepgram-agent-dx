# Runs: Condition A repeats (A2 to A5, Amendment 3)  |  Date: 2026-10-05 (US Eastern)

**Agent / model:** Claude Code CLI, Claude Sonnet 5.5 (all runs)
**Setup:** identical to Condition A (Amendment 1): new folder per run, same clip, `deepgram-sdk` uninstalled, `DEEPGRAM_API_KEY` set in the launching terminal, identical prompt. No docs pointer. Each agent's first command confirmed that no Deepgram SDK was installed and that the key was set.
**Source:** Claude Code session logs (timestamps in UTC)

## Results

| Run | Start (UTC) | Docs consulted | Diarization setting | Summarization | Tested with `curl` first | First successful call | Wall time | Errors | Interventions |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A (original, Oct 2) | 00:34:53 | None | `diarize=true` | `summarize=v2` on `/v1/listen` | Yes | 12s | 37s | 0 | 0 |
| A2 | 16:30:57 | None | `diarize=true` | `summarize=v2` on `/v1/listen` | Yes | 16s | 55s | 1 | 0 |
| A3 | 17:32:08 | None | `diarize=true` | `summarize=v2` on `/v1/listen` | Yes | 11s | 32s | 0 | 0 |
| A4 | 17:34:59 | None | `diarize=true` | `summarize=v2` on `/v1/listen` | No | 16s | 26s | 0 | 0 |
| A5 | 17:37:16 | None | `diarize=true` | `summarize=v2` on `/v1/listen` | Yes | 12s | 34s | 0 | 0 |

**Across all five runs:** no docs consulted in 5 of 5; `diarize=true` in 5 of 5; `summarize=v2` on `/v1/listen` in 5 of 5. Wall time ranged from 26 to 55 seconds (median 34); time to first successful call from 11 to 16 seconds (median 12).

## Run notes

- **A2:** a `sed` command failed on macOS (`invalid command code`); the agent made the edit with another tool and continued. Used raw HTTP (`requests`).
- **A3:** raw HTTP (`httpx`). Its final message noted that "the diarization split a few sentences across the wrong speaker," giving "Back" / "then, a dollar was a dollar" as an example.
- **A4:** wrote and ran the tool without a separate `curl` test. Raw HTTP (`requests`). Its final message said "a few words were assigned to the wrong speaker, for example 'Back' at the end of Speaker 0's first turn," and described the summary as "readable but a little clumsy."
- **A5:** Python standard library only. Its final message said "a few short replies were attributed to the wrong speaker," giving "Oh yeah." as an example, called the summary "short and sometimes imprecise," and offered to "add a second pass that sends the transcript to Claude for a better summary."

## What the agents told the developer, across all five runs

| Statement in the agent's final message | Runs |
| --- | --- |
| Some replies were assigned to the wrong speaker | A, A3, A4, A5 (4 of 5) |
| A limitation of the summary (for example, "fairly literal," "a little clumsy") | A, A2, A4, A5 (4 of 5) |
| A suggestion or offer to replace the summary with another LLM | A, A2, A5 (3 of 5) |

## Notes for the memo (interpretation)

- The baseline behavior was consistent across five runs on the dimensions this study scores. The timing was not.
- Four of five agents reported speaker misattributions without being asked, and three of five suggested an alternative to Deepgram's summarizer.
