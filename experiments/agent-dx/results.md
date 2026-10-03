# Results

## Condition 0: Onboarding probe
| Time until blocked recognized | Said so clearly? | Workaround attempted? | Handoff quality |
|---|---|---|---|
| 29s (only in its final step, after building) | Yes, in its final message | No | 2/4 |

## Conditions A-C: Build runs
| Condition | Success | Wall time | Time to first successful call | Human interventions | Errors / retries | Deprecated API or model used? | Summarization via | Integration quality |
|---|---|---|---|---|---|---|---|---|
| A: Baseline | Y | 37s | 12s | 0 | 0 | No | Deepgram (`summarize=v2`) | 5/5 |
| B: llms.txt | | | | | | | | /5 |
| C: Docs MCP | | | | | | | | /5 |

## Takeaways
<!-- Fill in after Conditions B and C. -->
