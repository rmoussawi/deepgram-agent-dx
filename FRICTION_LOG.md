# Friction Log

Log in the moment, not after. Record what happened, not just how it felt.

**Severity:** Blocker (could not proceed without help) · Major (wasted 10+ minutes or wrong path) · Minor (annoying, recovered quickly) · Observation (no direct friction, but relevant to agent experience)

**Actor:** Human (me) · Agent (coding agent)

| Time | Actor | Where (URL / step) | What happened | Severity | Suggested fix |
|---|---|---|---|---|---|
| 2026-10-01 23:05 ET | Agent | https://developers.deepgram.com/docs/summarization vs https://developers.deepgram.com/docs/models | Summarization lists support as "Nova"; the Models page labels "Nova" as legacy Nova 1. The agent couldn't tell whether `nova-3` supports summarization. (Condition A later showed it does: `nova-3` with `summarize=v2` returned a summary.) | Major | State supported models explicitly (e.g., "nova-3, nova-2, nova") in a compatibility table |
| 2026-10-01 23:05 ET | Agent | Python SDK type signature | Signature lists current and legacy models side by side with no feature compatibility, so the SDK couldn't resolve the question either | Minor | Expose feature-by-model support in SDK types or docstrings |
| 2026-10-02 | Human | https://developers.deepgram.com/docs/audio-intelligence | Code examples use different models by language: JavaScript and Java use `nova-3`; Python and Go use `nova-2`. An agent copying the Python or Go example would pick an older model. | Minor | Use the current recommended model in every language's example, and check the examples against each other automatically |
| 2026-10-01 23:05 ET | Agent | Docs fetch (via agent's summarizing fetch tool) | Agent reported the *supported* model as the *recommended* one; ambiguous wording gets distorted further when summarized | Observation | Unambiguous, plain statements of defaults and compatibility |
| 2026-10-02 11:19 PM ET | Agent | https://api.deepgram.com/v1/listen (Condition B) | `summarize=v2` with `nova-3` and `diarize_model=latest` returned HTTP 200 with no summary and no warning. The agent concluded summarization doesn't work with nova-3 and wrote that into its README (false: Condition A got a summary). | Major | Return an error or warning when a requested feature isn't applied |
| 2026-10-03 8:17 AM ET | Agent | Docs MCP server, https://developers.deepgram.com/_mcp/server (Condition C1) | `searchDocs` failed on every call: `Search failed: {"error":"Failed to fetch from FAI chat service"}`. Still failing at 8:45 AM and 12:33 PM ET (over 4 hours). The agent fell back to guessing page URLs, missed the Summarization page, and told the developer summarization exists only on `/v1/read`. | Major | Monitor agent-facing services like production APIs; fall back to static search when the chat service is down |

## Earlier website walkthrough (from memory)
TBD
## Milestones
| Milestone | Timestamp | Elapsed |
|---|---|---|
| Landed on deepgram.com | | |
| Account created | | |
| API key in hand | | |
| First successful API call | | |
