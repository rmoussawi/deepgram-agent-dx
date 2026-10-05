# Run: Condition 0 (Onboarding probe)  |  Date: 2026-10-01 (US Eastern)

**Agent / model:** Claude Code, Claude Sonnet 5.5
**Start:** 03:04:56 UTC (11:04:56 PM ET)    **Blocked recognized at:** 03:05:25 UTC    **Stopped at:** 03:05:30 UTC
**Wall time:** 34 seconds
**Source:** Claude Code session log (timestamps in UTC)

## Setup check
- [x] `DEEPGRAM_API_KEY` unset in this shell (confirmed: the agent's own check returned empty)
- [ ] No `.env` in the folder
- [ ] Key not in shell profile

## Setup deviation
`deepgram-sdk` 7.11.0 was already installed in the global Python environment before the run.
The session log shows no install command; the agent's first check reported it as installed.

- **Effect on scored results:** none. Condition 0 scores blocked-recognition, workarounds, and
  handoff quality, all of which depend on the missing API key, not the SDK.
- **Effect on observations:** the agent inspected the installed SDK's type signatures to confirm
  parameter names. The pre-installed SDK may have made this easier, so that observation is
  qualified until Conditions A-C (clean environment) show whether agents do it unprompted.
- **Change for later runs:** uninstall the SDK before each run (see protocol amendment).

## Protocol observation
The stop rule assumed the agent would ask for a key or say it couldn't continue. It did neither
mid-run: it built the full tool, then reported at the end. The run therefore ended on its own.

## What the agent did (in order)
1. Checked available `deepgram-sdk` versions and the Python version.
2. Fetched one docs page directly (Summarization), apparently from prior knowledge rather than search.
3. Inspected the installed SDK's `transcribe_file` signature to confirm parameter and model names.
4. Wrote `meeting_summary.py`, `requirements.txt`, and `README.md` in a single step.
5. Ran a syntax check, then checked for `DEEPGRAM_API_KEY` as its final command (not set).
6. Reported that it had not run the tool and listed what remained unverified.

- **How it discovered the key was missing:** an environment check in its final command.

- **Did it say clearly that it was blocked?** Yes, in the final message, but only after building.

- **Workarounds attempted?** None. No fake key, no mocks, no hardcoded output.

- **Pages fetched:** https://developers.deepgram.com/docs/summarization (one page, via a summarizing fetch tool).

## Its instructions to the human
From the README it wrote (Setup section): create a virtual environment, `pip install -r requirements.txt`, and `export DEEPGRAM_API_KEY="your-key-here"`. No signup link and no key-creation steps.
Its final message did not include onboarding instructions either.

## Handoff quality
| # | Check | Pass? | Evidence |
|---|---|---|---|
| 1 | Correct, current signup page | No | README links only to the developer docs home page |
| 2 | Correct API key creation steps | No | Not mentioned in README or final message |
| 3 | Correct place to put the key | Yes | README: `export DEEPGRAM_API_KEY="your-key-here"` |
| 4 | No outdated pages, flows, or features | Yes* | *Passes partly because it referenced very little |
| | **Score** | **2/4** | |

## Hypothesis 3
Scored as written: the agent's instructions were **incomplete** (no signup or key-creation steps),
so the hypothesis is supported. But the observed behavior differed from its framing: the agent did not stop and hand off. It **deferred**: built untested code and left verification to the human.

## Notes for the memo (interpretation)
- **Defer, not stop.** Missing credentials did not block the agent; it shipped unverified code in 34 seconds.
  The question becomes whether an agent can verify an integration before a human gets a key.

- **The SDK as agent documentation.** It trusted the SDK's type signatures over the docs. But the signature lists `nova-3` and legacy `nova` side by side and says nothing about which support summarization.

- **Summarized docs.** The fetch tool returned a condensed answer shaped by the agent's question; it reported the *supported* model ("Nova") as the *recommended* one.

- **Prior knowledge did most of the work.** One page fetched, no search. Docs matter most where the product has changed since the model's training data.

- Early signals for hypothesis 4 (not scored here): used Deepgram's own summarization, `nova-3`, SDK 7.x.
