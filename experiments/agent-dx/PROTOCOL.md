# Agent DX Experiment: Protocol

## Question
How easily can an AI coding agent discover, integrate, and correctly use Deepgram,
and how much do Deepgram's agent-facing docs (llms.txt, docs MCP server) help?

## Conditions
Each condition is run in a **fresh session, in a new empty folder**, with the same agent, same model, and the same prompt.

| ID | Condition | Setup |
|---|---|---|
| 0 | Onboarding probe | **No API key.** Same prompt, but DEEPGRAM_API_KEY is unset. Measures the handoff, not the build. |
| A | Baseline | Agent has web search only. No Deepgram hints. |
| B | llms.txt | Same prompt, plus: "Deepgram's docs index is at https://developers.deepgram.com/llms.txt" |
| C | Docs MCP | Deepgram docs MCP server connected (https://developers.deepgram.com/_mcp/server). Same prompt as A. |

Run order: 0, then A, B, C. Run 0 and A first. That keeps us from unintentionally steering the agent.

### Condition 0 (original prompt)
> Build a Python command-line meeting-summary tool. Given a path to an audio file of a meeting, it should transcribe the audio using Deepgram, label different speakers, and print a short summary of the meeting.
> The Deepgram API key is in the environment variable DEEPGRAM_API_KEY. Use current, recommended Deepgram APIs and models. Include a README explaining how to run it.

### Conditions A-C (original prompt plus the Amendment 1 sentence)
We use this exact text for A, B, and C. For B, add the llms.txt line described in the conditions table.
> Build a Python command-line meeting-summary tool. Given a path to an audio file of a meeting, it should transcribe the audio using Deepgram, label different speakers, and print a short summary of the meeting.
> The Deepgram API key is in the environment variable DEEPGRAM_API_KEY. Use current, recommended Deepgram APIs and models. Include a README explaining how to run it.
> A sample meeting recording for testing is at ./meeting.wav.

The summary method is deliberately unspecified. We note whether the agent finds Deepgram's own summarization feature or reaches for a separate LLM. That is a discovery signal.

## Rules
- We only provide the API key, approve tool permissions, and answer direct questions the agent asks.
- Every other intervention counts as a **human intervention** and gets logged.
- We stop a run at 45 minutes or 3 consecutive failed fixes. A stopped run counts as a fail.
- Test audio: a public-domain or self-recorded clip with 2+ speakers.

## Condition 0: Onboarding probe
The prompt is identical, including the sentence saying the key is in DEEPGRAM_API_KEY. That sentence is deliberately false here, which mirrors a real developer who hasn't finished setup.

**Setup:** confirm the key is not available anywhere the agent can see:
`unset DEEPGRAM_API_KEY`, no `.env` file in the folder, and the key not set in your shell profile.

**Rules:**
- We stop the run when the agent asks for a key or says it can't continue, or at 20 minutes.
- We don't create a key or give hints during this run.
- If the agent works around the block (fake key, mocked Deepgram responses, hardcoded sample output) instead of asking, we record it. That is a finding, not a failure of the run.

**Handoff quality checklist (1 point each, out of 4).** To verify against Deepgram's current site and docs:
1. Points to the correct, current signup page.
2. Gives correct steps to create an API key in the console.
3. Says correctly where to put the key (the env var the code expects).
4. Doesn't reference outdated pages, flows, or features.

We also record: time until the agent recognized it was blocked, and whether it said so clearly
or tried to keep going.

## Integration quality checklist, Conditions A-C (score 1 point each, out of 5)
This measures whether the agent used Deepgram correctly, not how good the summary prose is.
Summary quality depends mostly on whichever LLM writes it, so it is out of scope.

1. Uses a current, recommended speech-to-text model (not deprecated).
2. Speaker labels (diarization) are enabled **and** the output actually separates the speakers.
3. Uses current SDK or API patterns, not legacy code copied from outdated pages.
4. Uses Deepgram's own features where they exist (e.g., summarization) instead of rebuilding them.
5. Handles a missing or invalid API key with a clear error message.

We verify each item against Deepgram's current docs at scoring time, and note the source checked.

## What to record per run
We use `runs/RUN_TEMPLATE.md`. The agent's transcript or log is saved.

## Amendment 1 (2026-10-02, before Condition A)
Condition 0 surfaced three setup gaps. These changes apply to Conditions A-C only. Condition 0's results stand as recorded.

**1. Test audio.** Condition 0 showed the agent can't test end to end without a sample file.
A test clip is placed in each run folder, and this sentence is appended to the task prompt:
(the full A-C prompt is shown in the task prompt section above):
> A sample meeting recording for testing is at ./meeting.wav.

A, B, and C use the identical prompt and the identical clip: 2-3 minutes, at least two speakers,
English, well over 50 words of speech, and either self-recorded with consent or public-domain.

**2. Clean environment.** In Condition 0, `deepgram-sdk` was already installed globally before the run.
Before each of A-C:
- `pip3 uninstall -y deepgram-sdk`
- Confirm with `python3 -c "import deepgram"`, which should fail with `ModuleNotFoundError`.

**3. Neutral folder names.** Condition 0's folder name described the experiment. Runs use
`~/dx-runs/run-a`, `run-b`, `run-c`, each new and empty except for the test clip.

**Definition: first successful call.** The first time code written by the agent receives a
transcript from Deepgram (not just an authenticated connection or HTTP 200). Measured from prompt submission, using session-log timestamps.

**Permissions.** We approve permission prompts immediately and the same way in every run, so approval time doesn't distort timing differences between conditions.

**API key.** For A-C, we export the key only in the terminal that launches the agent
(`export DEEPGRAM_API_KEY=...`), never in a file inside the run folder or in a message to the agent.

