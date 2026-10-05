# Runs: Selection probe (S1 to S5, Amendment 3)  |  Date: 2026-10-05 (US Eastern)

**Agent / model:** Claude Code CLI, Claude Sonnet 5.5 (all runs)
**Setup:** new folder per run, same clip. The Deepgram, Gemini, OpenAI, and Anthropic keys were unset in the launching terminal, and none of the variable names in the agents' first environment-variable searches were set. The prompt did not name a provider (see Amendment 3 in [`PROTOCOL.md`](../PROTOCOL.md)).
**Stop rule:** when the agent had chosen a provider and either asked for a key or started building, or at 10 minutes. In every run, the agent finished on its own.
**Source:** Claude Code session logs (timestamps in UTC)

## Results

| Run | Start (UTC) | Names in its first environment-variable search | Speech-to-text and speaker labels | Summary | Docs consulted | Time |
| --- | --- | --- | --- | --- | --- | --- |
| S1 | 17:52:58 | `ASSEMBLY\|DEEPGRAM\|OPENAI\|ANTHROPIC` | AssemblyAI (raw HTTP) | Claude | None | 21s |
| S2 | 17:58:58 | `ASSEMBLY\|DEEPGRAM\|OPENAI\|ANTHROPIC\|HF_` | AssemblyAI (Python SDK) | Claude | None | 24s |
| S3 | 18:02:55 | `ASSEMBLY\|DEEPGRAM\|ANTHROPIC\|OPENAI\|GOOGLE_APP` | AssemblyAI (Python SDK) | Claude | None | 27s |
| S4 | 18:06:16 | `ASSEMBLY\|DEEPGRAM\|OPENAI\|ANTHROPIC\|GOOGLE\|AWS_` | AssemblyAI (Python SDK) | Claude | None | 32s |
| S5 | 18:10:55 | `ASSEMBLY\|DEEPGRAM\|ANTHROPIC\|OPENAI\|ELEVEN\|HF_TOKEN` | AssemblyAI (Python SDK) | Claude | None | 26s |

**Across all five runs:** AssemblyAI was chosen for speech-to-text and speaker labels in 5 of 5, and Claude for the summary in 5 of 5. Deepgram was chosen in 0 of 5; its name appeared in the first environment-variable search in 5 of 5 (the search is the agent's first `env | grep -iE '...'` command, shown in the table). No docs were consulted in any run. Every tool required two API keys (AssemblyAI and Anthropic).

## The reason each agent gave for choosing AssemblyAI

| Run | Reason, as stated in the agent's final message |
| --- | --- |
| S1 | "AssemblyAI does both in one call, so there's no separate diarization step." |
| S2 | "AssemblyAI, with speaker labels turned on. It does both in one upload and accepts wav, mp3, m4a and most other formats." |
| S3 | "AssemblyAI does both in one call. It's a single hosted API that handles speaker labelling well, and it takes common audio formats directly, so there's no ffmpeg step." |
| S4 | "One call returns the transcript with speaker labels, so there is no separate diarization step to set up." |
| S5 | "AssemblyAI does both in one call, so there's no separate diarization step." |

## What each agent reported doing without keys

No run stopped partway to ask for keys. Each built the tool, and its final message listed the two keys for the developer to set. What each agent said it checked, from its final message:

- **S1:** syntax check only. Its final message noted it had written AssemblyAI's model names "from memory" and that they might need correcting.
- **S2:** installed dependencies, checked `--help`, and confirmed a clear error for missing keys. Offered to "swap the summary step to AssemblyAI's own LLM endpoint" to use one vendor.
- **S3:** compiled the script, installed both packages, and confirmed clear errors for a missing key or file.
- **S4:** checked imports, `--help`, and errors for a missing file or key, and confirmed the SDK accepts its configuration options.
- **S5:** checked imports and argument parsing, and that it stops at the missing-key check.

## Notes for the memo (interpretation)

- When the task didn't name a provider, this agent and model chose the same speech-to-text provider in every run, without consulting docs. Deepgram's share of choices in this probe was 0 of 5.
- The reason given in every run, transcription and speaker labels from one request, also describes Deepgram's API as used in this study.
- This is one agent, one model, and one task, so it says nothing yet about other agents or tasks.
