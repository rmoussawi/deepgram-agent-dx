# Results

New to these tables? See [How to read these tables](#how-to-read-these-tables) at the end for what each field means, with examples from the runs.

## Condition 0: Onboarding probe
| Time until blocked recognized | Said so clearly? | Workaround attempted? | Handoff quality |
|---|---|---|---|
| 29s (only in its final step, after building) | Yes, in its final message | No | 2/4 |

## Conditions A-C: Build runs
| Condition | Success | Wall time | Time to first successful call | Human interventions | Errors / retries | Deprecated API or model used? | Summarization via | Integration quality |
|---|---|---|---|---|---|---|---|---|
| A: Baseline | Y | 37s | 12s | 0 | 0 | Yes: `diarize=true` (deprecated) | Deepgram (`summarize=v2` on `/listen`) | 4/5 |
| B: llms.txt | Y | 70s | 29s | 0 | 0 errors; 1 silent failure (no summary from `/listen`) | No | Deepgram (`/v1/read` Text Intelligence) | 5/5 |
| C1: Docs MCP (not administered: MCP outage) | Y | 75s | 49s | 0 | 2 MCP errors (docs search down); 0 API errors | No | Deepgram (`/v1/read` Text Intelligence) | 5/5 |
| C2: Docs MCP (rerun) | | | | | | | | /5 |

## Output comparison: A vs. B (same clip, same model)
Not scored; recorded because it connects integration choices to what the developer sees.

- **Transcription:** word-for-word the same, including the same errors ("Sashfak," "Sassafrac" for sassafras). Expected: both used `nova-3`.
- **Speaker attribution:** clearly better in B. A (deprecated `diarize=true`, v1 diarizer) put several replies under the wrong speaker: "Oh yeah," "Well, boil it and you drink it," and "Did that help you?" all landed in the wrong turn, and a stray "Back" ended Speaker 0's opening question. B (`diarize_model=latest`, v2 diarizer) split those correctly, with one mid-sentence split at [01:16].
  - *Caveat:* B's code also absorbs one-word speaker flips. That can't explain most of the difference, since A's errors are mostly multi-word, but the two effects aren't fully separated. One clip only.
- **Summaries:** both short and imprecise. B's summary, made by sending the labelled transcript to `/v1/read`, reversed who asked the opening question. Summary quality is out of scope for scoring.

**Why it matters:** the developer in Condition A would see weaker speaker labels and could reasonably judge Deepgram's diarization by them, without knowing a better diarizer was one parameter away.

C1 is shown for completeness but isn't a valid Condition C result: the docs MCP server failed on every call, so the agent fell back to fetching pages directly. See `runs/condition-c1.md`.

## Takeaways
<!-- Fill in after Conditions B and C. -->

## How to read these tables
All times come from the Claude Code session logs and are measured from the moment the prompt was submitted.
Each condition was run once (see Limitations in [`FINDINGS.md`](../../FINDINGS.md)).

### Condition 0: Onboarding probe
The agent got the same task with **no API key**. These fields measure what it did at the step it couldn't complete itself, not the quality of the code.

| Field | What it means | Example from the runs |
|---|---|---|
| Time until blocked recognized | Time until the agent noticed it had no key and said so | 29s. It noticed only in its last step, after building the whole tool |
| Said so clearly? | Whether the agent told the developer plainly that it couldn't finish, rather than implying success | Yes, in its final message: it hadn't run the tool because the key wasn't set |
| Workaround attempted? | Whether it tried to fake success instead: a made-up key, mocked responses, or hardcoded output | No. Counts as a finding if it happens, not a failure of the run |
| Handoff quality (x/4) | How well its instructions to the human cover the missing step: signup page, key creation, where to put the key, and no outdated references. 1 point each | 2/4. It said where to put the key but not how to sign up or create one |

### Conditions A-C: Build runs
The same task and prompt, with a key, under different documentation setups:
**A** no help beyond what the agent already knows or searches for, **B** pointed to Deepgram's `llms.txt`, **C** connected to Deepgram's docs MCP server.

| Field | What it means | Example from the runs |
|---|---|---|
| Condition | Which documentation setup the agent had | B: the prompt included a line pointing to `llms.txt` |
| Success | Whether the finished tool works end to end (transcript, speaker labels, and summary), checked by me running it, not by the agent's claim. Y / N / Partial | Y in A, B, and C1 |
| Wall time | Total time from prompt to the agent's final message | 37s in A; 70s in B |
| Time to first successful call | Time until code written by the agent first received a transcript from Deepgram. An authenticated connection or HTTP 200 alone doesn't count | 12s in A, from a quick `curl` test before writing the tool |
| Human interventions | Anything I did beyond giving the prompt, providing the key, approving permissions, or answering direct questions | 0 in every run |
| Errors / retries | Failures the agent hit and how many times it changed approach. Includes **silent failures**: a success response missing what was requested | B: 0 errors, 1 silent failure (HTTP 200 but no summary) |
| Deprecated API or model used? | Whether the code relies on anything Deepgram's current docs mark as deprecated or legacy, checked against the live docs | A: yes, `diarize=true`, which the Diarization page says to replace with `diarize_model` |
| Summarization via | Where the summary came from: Deepgram's audio summarization (`summarize` on `/v1/listen`), Deepgram's text summarization (`/v1/read`), or a separate LLM | A used `/v1/listen`; B and C1 used `/v1/read` |
| Integration quality (x/5) | Whether the agent used Deepgram correctly, scored with the five-point checklist in [`PROTOCOL.md`](PROTOCOL.md): current model, working speaker labels, current API patterns, Deepgram's own features, and a clear missing-key error. It does not grade how good the summary reads | A: 4/5, losing a point for the deprecated diarization parameter |

### Terms used in the tables
- **Not administered:** the condition couldn't be applied as designed, so the run isn't a valid result for it.
  C1 is marked this way because the docs MCP server was down for every call.
- **Silent failure:** the API reports success but leaves out something that was requested, with no error or warning. Example: B's request asked for a summary and got HTTP 200 without one.
- **Diarization:** labeling who spoke when (Speaker 0, Speaker 1, ...).
- **`/v1/listen` and `/v1/read`:** Deepgram's endpoints for audio (transcription, plus features like summarization) and for text analysis (summarizing text you send it).

