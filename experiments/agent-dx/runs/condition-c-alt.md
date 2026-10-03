# Run: Condition C-alt (Alternate docs MCP, Amendment 2)  |  Date: 2026-10-03 (US Eastern)

**Agent / model:** Claude Code CLI, Claude Sonnet 5.5
**Server:** `https://api.dx.deepgram.com/kapa/mcp` (listed on Deepgram's Agentic developer tools page), tool `search_deepgram_knowledge_sources`
**Start:** 19:09:54 UTC (3:09:54 PM ET)    **First successful call:** 19:10:33 UTC    **End:** 19:10:46 UTC
**Wall time:** 52 seconds    **Time to first successful call:** 39 seconds
**Source:** Claude Code session log (timestamps in UTC)

## Setup check (Amendments 1 and 2)
- [x] New, empty folder with a neutral name (`~/Code/dx-runs/run-c-alt`), containing only `meeting.wav`
- [x] Same test clip as A-C
- [x] `deepgram-sdk` uninstalled globally (the agent installed 7.12.0 into its own virtual environment)
- [x] `DEEPGRAM_API_KEY` exported in this terminal only (log confirms: agent's check returned "yes")
- [x] Prompt identical to A (no llms.txt line, no mention of MCP)
- [x] Server added for this folder only (no `--scope project`)
- [x] One-time browser sign-in completed before the run. The page offered sign-in through a popular email  provider or with an email address. Any Google account worked, with no existing Deepgram account needed;  Google asked permission to share basic account information. It took about 22 seconds, and can take a few seconds if you're already signed in to the provider in the browser.
  The Agentic developer tools page doesn't mention that sign-in is required.

### Connection problems before the run (setup, not interventions)
| Time (UTC) | What happened |
|---|---|
| <!-- time --> | `/mcp` showed the server connected, but no tools were listed. Restarted Claude Code |
| 19:02:52 | Reconnect failed: "Version negotiation probe timed out after 5000ms" |
| 19:05:41 | Reconnected; `search_deepgram_knowledge_sources` available |

## Outcome
- Working end to end (transcribe + speaker labels + meeting summary)? **Y** (confirmed by my own run with the key set)
- Human interventions: 0
- Errors hit and retries: 0

## Timeline
| Time (UTC) | Elapsed | Event |
|---|---|---|
| 19:09:54 | 0s | Prompt submitted |
| 19:09:57 | 3s | Checked folder, Python, and key; loaded the docs MCP search tool on its own |
| 19:09:59 | 5s | Two searches in parallel: diarization with the Python SDK, and summarization |
| 19:10:07 | 13s | Results returned (about 8 seconds per search) |
| 19:10:14 | 20s | Created a virtual environment, installed `deepgram-sdk` 7.12.0, inspected the SDK signature |
| 19:10:27 | 33s | Wrote `meeting_summary.py` and `requirements.txt` |
| 19:10:33 | 39s | **First successful call** (via its own tool): 2 speakers, summary, transcript |
| 19:10:41 | 47s | Wrote `README.md`; tested missing-file and missing-key errors |
| 19:10:46 | 52s | Final report |

## Discovery
- **Did the agent use the MCP server unprompted?** Yes, as its first docs step.
- **What the searches returned:** 28 results across two searches, mostly from GitHub: Deepgram's `recipes` repo, SDK examples, and community discussions. For the diarization search, only one of 13 results came from developers.deepgram.com (the API reference). **The Diarization docs page never appeared,** and no result mentioned `diarize_model` or the deprecation of `diarize=true`.
- SDK or raw HTTP? **Official Python SDK** (`deepgram-sdk` 7.12.0), the only build run to use it.
- Models / endpoints chosen: `nova-3` on `/v1/listen`, with the **deprecated** `diarize=true`.
- Summarization: **Deepgram's audio summarization** (`summarize="v2"` on `/v1/listen`), found through the recipes.

## Search results (from the session log)
The agent ran two searches with the server's `search_deepgram_knowledge_sources` tool. Below are the top six results for each, in the order returned.

**Search 1:** *"How do I transcribe a prerecorded audio file with speaker diarization using the Deepgram Python SDK and the latest Nova model?"*

| # | Source | Type |
|---|---|---|
| 1 | github.com/deepgram/recipes/.../python/speech-to-text/v1/diarize/example.py | Code recipe |
| 2 | github.com/orgs/deepgram/discussions/501 | Community discussion |
| 3 | github.com/deepgram/deepgram-python-sdk/.../.agents/skills/deepgram-python-audio-intelligence/SKILL.md | SDK agent skill |
| 4 | Same file as #3, a different section | SDK agent skill |
| 5 | github.com/deepgram/recipes/.../python/speech-to-text/v1/diarize/README.md | Code recipe |
| 6 | github.com/deepgram/deepgram-python-sdk/.../examples/11-transcription-prerecorded-file.py | SDK example |

The top result, Deepgram's Python diarization recipe, uses the deprecated parameter:

```python
response = client.listen.v1.media.transcribe_url(
    url=AUDIO_URL,
    model="nova-3",
    smart_format=True,
    diarize=True,      # <-- THIS is the feature this recipe demonstrates.
```

Across all 13 results for this search, only one came from developers.deepgram.com (the API reference). The
Diarization docs page, which says to replace `diarize=true` with `diarize_model`, wasn't among them.

**Search 2:** *"How do I use the Deepgram summarize or text intelligence features to get a summary of a prerecorded transcript?"*

| # | Source | Type |
|---|---|---|
| 1 | github.com/deepgram/recipes/.../go/audio-intelligence/v1/summarize/example.go | Code recipe |
| 2 | developers.deepgram.com/docs/text-summarization | Docs page |
| 3 | developers.deepgram.com/docs/summarization | Docs page |
| 4 | github.com/deepgram/recipes/.../go/speech-to-text/v1/summarize/example.go | Code recipe |
| 5 | github.com/deepgram/recipes/.../javascript/speech-to-text/v1/summarize/example.js | Code recipe |
| 6 | github.com/deepgram/recipes/.../rust/audio-intelligence/v1/summarize/src/main.rs | Code recipe |

**The contrast:** the summarization search returned both official Summarization pages near the top, and the agent found the right approach. The diarization search returned no Diarization page, and the agent followed an outdated recipe. The server does return docs pages; for diarization, the recipe ranked first and the page explaining the change didn't appear.

## Integration quality
| # | Check | Pass? | Evidence |
|---|---|---|---|
| 1 | Current STT model | Yes | `nova-3` |
| 2 | Diarization on and working | Yes | `diarize=True`; 2 speakers separated (with the same misattributions as Condition A) |
| 3 | Current SDK/API patterns | No | Current SDK (7.12.0), but used `diarize=True`, which https://developers.deepgram.com/docs/diarization marks deprecated |
| 4 | Uses Deepgram's own features | Yes | `summarize="v2"` on `/v1/listen` |
| 5 | Clear API key error handling | Yes | Confirmed by my own run with the key unset; the agent also tested it |
| | **Score** | **4/5** | |

## Where it went wrong
- **The recommended docs server pointed the agent to a deprecated parameter.** Deepgram's own diarization recipe uses `diarize`, and the search never surfaced the Diarization page that explains its replacement. The output shows it: speaker attribution is identical to Condition A's, including the same errors, because both used the older diarizer.

## Notes for the memo
- **The best summarization result, but stale diarization.** Of the three docs-assisted runs, only C-alt found audio summarization on `/v1/listen`, because the recipes show it. But the same recipes carried the deprecated diarization parameter.
- **What a docs search indexes matters as much as whether it works.** This server returned mostly code recipes, examples, and forum posts, not docs pages. Those are useful for agents, but they go out of date unless they're maintained alongside the docs.
- **Fastest docs-assisted run.** 52s total, versus 70s (B) and 75s (C1). Two searches replaced several page fetches.
- **Reaching the server took effort.** A browser sign-in the setup page doesn't mention, a connection with no tools, a timeout, then success after a retry.
