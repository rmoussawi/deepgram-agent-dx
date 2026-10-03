# Run: Condition __  |  Date/time: ____

**Agent / model:**
**Start:**        **First successful call:**        **End:**        **Wall time:**

## Setup check (Amendment 1)
- [ ] New, empty folder with a neutral name (e.g., `~/dx-runs/run-a`)
- [ ] Test clip in folder: `meeting.wav` (same clip for A-C)
- [ ] `deepgram-sdk` uninstalled; `python3 -c "import deepgram"` fails
- [ ] `DEEPGRAM_API_KEY` exported in this terminal only
- [ ] Prompt includes the amendment sentence about `./meeting.wav`
- [ ] Condition-specific setup done (B: llms.txt line added; C: docs MCP connected)

## Outcome
- Working end to end (transcribe + speaker labels + meeting summary)? Y / N / Partial
- Human interventions (count, and what each was):
- Errors hit and retries:

## Discovery
- First Deepgram source the agent consulted:
- Doc pages / URLs fetched (in order):
- SDK or raw HTTP? Which SDK version?
- Models / endpoints chosen. Current or deprecated?
- Summarization: Deepgram feature or another LLM?

## Integration quality (see PROTOCOL.md)
| # | Check | Pass? | Evidence |
|---|---|---|---|
| 1 | Current STT model | | |
| 2 | Diarization on and working | | |
| 3 | Current SDK/API patterns | | |
| 4 | Uses Deepgram's own features | | |
| 5 | Clear API key error handling | | |
| | **Score** | **/5** | |

## Where it went wrong
<!-- Exact error messages, wrong assumptions, stale snippets it copied. -->

## Notes for the memo
<!-- One or two observations worth a stake-in-the-ground opinion. -->
