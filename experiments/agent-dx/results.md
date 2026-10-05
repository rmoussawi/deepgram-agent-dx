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
| B: llms.txt | Y | 70s | 29s | 0 | 0 errors; 1 misread response (missed `results.summary`) | No | Deepgram (`/v1/read` Text Intelligence) | 5/5 |
| C1: Docs MCP (not administered: MCP outage) | Y | 75s | 49s | 0 | 2 MCP errors (docs search down); 0 API errors | No | Deepgram (`/v1/read` Text Intelligence) | 5/5 |
| C-alt: Alternate docs MCP (Amendment 2) | Y | 52s | 39s | 0 | 0 | Yes: `diarize=true` (deprecated) | Deepgram (`summarize=v2` on `/listen`) | 4/5 |

## Output comparison: A vs. B (same clip, same model)
Not scored; recorded because it connects integration choices to what the developer sees.

- **Transcription:** word-for-word the same, including the same errors ("Sashfak," "Sassafrac" for sassafras).
  Expected: both used `nova-3`.
- **Speaker attribution:** clearly better in B. A put several replies under the wrong speaker: "Oh yeah," "Well, boil it and you drink it," and "Did that help you?" all landed in the wrong turn, and a stray "Back" ended Speaker 0's opening question. B split those correctly, with one mid-sentence split at [01:16].
  - *Why:* not the diarizer setting, as first assumed. C1 used the same setting as B (`diarize_model=latest`) and produced speaker turns identical to A's. The runs differ in how they built speaker turns: A, C1, and C-alt used Deepgram's utterance segments, while B grouped individual words by speaker and merged one-word flips. Whether that difference caused the different labels wasn't tested. One clip.
- **Summaries:** both short and imprecise. B's summary, made by sending the labelled transcript to `/v1/read`,
  reversed who asked the opening question. Summary quality is out of scope for scoring.

**Why it matters:** the same API output can look better or worse depending on how the agent's code uses it. A developer judging Deepgram's diarization from A's tool would see weaker speaker labels than B's, from the same audio and service.

## Output comparison: C-alt and C1 vs. A (same clip, same model)
Not scored; recorded for the same reason as above.

- **C-alt: identical to A.** Transcript, speaker turns, and summary match Condition A's exactly, line for line, including every misattributed reply and the stray "Back." Both used the deprecated `diarize=true`, `summarize=v2` on `/v1/listen`, and Deepgram's utterance segments for speaker turns.
- **C1: same speaker turns as A.** C1 used the current `diarize_model=latest`, yet its transcript and speaker turns match A's line for line, including the same misattributions. Like A, it built speaker turns from utterance segments. Its summary differs, because it came from `/v1/read`.
- **What this shows:** on this clip, the diarizer setting didn't visibly change the speaker labels. The one run with better labels, B, is also the only one that built speaker turns from individual words. See the A vs. B section.

C1 is shown for completeness but isn't a valid Condition C result: the docs MCP server failed on every call, so the agent fell back to fetching pages directly. See `runs/condition-c1.md`. Condition C was not re-run: the server's search was still failing at the last check (12:50 PM ET on 2026-10-04). Instead, C-alt tested Deepgram's other documented docs MCP server (Amendment 2 in `PROTOCOL.md`). C-alt is reported separately and doesn't replace
Condition C.

## Repeat runs: Condition A (Amendment 3)
The same setup as Condition A, run four more times. Details: [`runs/condition-a-repeats.md`](runs/condition-a-repeats.md).

| Run | Docs consulted | Diarization setting | Summarization | First successful call | Wall time | Interventions |
| --- | --- | --- | --- | --- | --- | --- |
| A (original) | None | `diarize=true` | `summarize=v2` on `/listen` | 12s | 37s | 0 |
| A2 | None | `diarize=true` | `summarize=v2` on `/listen` | 16s | 55s | 0 |
| A3 | None | `diarize=true` | `summarize=v2` on `/listen` | 11s | 32s | 0 |
| A4 | None | `diarize=true` | `summarize=v2` on `/listen` | 16s | 26s | 0 |
| A5 | None | `diarize=true` | `summarize=v2` on `/listen` | 12s | 34s | 0 |

5 of 5 consulted no docs and used `diarize=true`. Wall time ranged from 26 to 55 seconds (median 34).

## Selection probe (Amendment 3)
The task without naming a provider, with the Deepgram, Gemini, OpenAI, and Anthropic keys unset. Details: [`runs/selection-probe.md`](runs/selection-probe.md).

| Run | Speech-to-text and speaker labels | Summary | Docs consulted | Time |
| --- | --- | --- | --- | --- |
| S1 | AssemblyAI | Claude | None | 21s |
| S2 | AssemblyAI | Claude | None | 24s |
| S3 | AssemblyAI | Claude | None | 27s |
| S4 | AssemblyAI | Claude | None | 32s |
| S5 | AssemblyAI | Claude | None | 26s |

No run chose Deepgram. In all five, the agent's first command searched the environment for variable names that included `DEEPGRAM`.

## Takeaways
See [`FINDINGS.md`](../../FINDINGS.md) for the analysis.

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
| Errors / retries | Failures the agent hit and how many times it changed approach, including its own mistakes, such as misreading a response | B: 0 errors, 1 misread response (it looked for the summary in the wrong place in the JSON) |
| Deprecated API or model used? | Whether the code relies on anything Deepgram's current docs mark as deprecated or legacy, checked against the live docs | A: yes, `diarize=true`, which the Diarization page says to replace with `diarize_model` |
| Summarization via | Where the summary came from: Deepgram's audio summarization (`summarize` on `/v1/listen`), Deepgram's text summarization (`/v1/read`), or a separate LLM | A used `/v1/listen`; B and C1 used `/v1/read` |
| Integration quality (x/5) | Whether the agent used Deepgram correctly, scored with the five-point checklist in [`PROTOCOL.md`](PROTOCOL.md): current model, working speaker labels, current API patterns, Deepgram's own features, and a clear missing-key error. It does not grade how good the summary reads | A: 4/5, losing a point for the deprecated diarization parameter |

### Terms used in the tables
- **Not administered:** the condition couldn't be applied as designed, so the run isn't a valid result for it.
  C1 is marked this way because the docs MCP server was down for every call.
- **Misread response:** the API returned what was requested, but the agent looked in the wrong place and concluded it was missing. Example: in B, the summary was under `results.summary`; the agent checked the top level.
- **Diarization:** labeling who spoke when (Speaker 0, Speaker 1, ...).
- **`/v1/listen` and `/v1/read`:** Deepgram's endpoints for audio (transcription, plus features like
  summarization) and for text analysis (summarizing text you send it).

