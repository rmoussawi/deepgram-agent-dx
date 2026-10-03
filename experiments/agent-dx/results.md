# Results

## Condition 0: Onboarding probe
| Time until blocked recognized | Said so clearly? | Workaround attempted? | Handoff quality |
|---|---|---|---|
| 29s (only in its final step, after building) | Yes, in its final message | No | 2/4 |

## Conditions A-C: Build runs
| Condition | Success | Wall time | Time to first successful call | Human interventions | Errors / retries | Deprecated API or model used? | Summarization via | Integration quality |
|---|---|---|---|---|---|---|---|---|
| A: Baseline | Y | 37s | 12s | 0 | 0 | Yes: `diarize=true` (deprecated) | Deepgram (`summarize=v2` on `/listen`) | 4/5 |
| B: llms.txt | Y | 70s | 29s | 0 | 0 errors; 1 silent failure (no summary from `/listen`) | No | Deepgram (`/v1/read` Text Intelligence) | 5/5 |
| C: Docs MCP | | | | | | | | /5 |

## Output comparison: A vs. B (same clip, same model)
Not scored; recorded because it connects integration choices to what the developer sees.

- **Transcription:** word-for-word the same, including the same errors ("Sashfak," "Sassafrac" for sassafras).
  Expected: both used `nova-3`.
- **Speaker attribution:** clearly better in B. A (deprecated `diarize=true`, v1 diarizer) put several replies
  under the wrong speaker: "Oh yeah," "Well, boil it and you drink it," and "Did that help you?" all landed in the wrong turn, and a stray "Back" ended Speaker 0's opening question. B (`diarize_model=latest`, v2 diarizer) split those correctly, with one mid-sentence split at [01:16].
  - *Caveat:* B's code also absorbs one-word speaker flips. That can't explain most of the difference, since A's errors are mostly multi-word, but the two effects aren't fully separated. One clip only.
- **Summaries:** both short and imprecise. B's summary, made by sending the labelled transcript to `/v1/read`, reversed who asked the opening question. Summary quality is out of scope for scoring.

**Why it matters:** the developer in Condition A would see weaker speaker labels and could reasonably judge Deepgram's diarization by them, without knowing a better diarizer was one parameter away.

## Takeaways
<!-- Fill in after Conditions B and C. -->
