# Friction Log

Entries were logged as they happened, except the section marked "from memory." Each records what happened.

**Severity:** Blocker (could not proceed without help) · Major (wasted 10+ minutes or wrong path) · Minor (annoying, recovered quickly) · Observation (no direct friction, but relevant to agent experience)

**Actor:** Human (me) · Agent (coding agent)

| Time | Actor | Where (URL / step) | What happened | Severity | Suggested fix |
|---|---|---|---|---|---|
| 2026-10-01 23:05 ET | Agent | https://developers.deepgram.com/docs/summarization vs https://developers.deepgram.com/docs/models | Summarization lists support as "Nova"; the Models page labels "Nova" as legacy Nova 1. The agent couldn't tell whether `nova-3` supports summarization. (Condition A later showed it does: `nova-3` with `summarize=v2` returned a summary.) | Major | State supported models explicitly (e.g., "nova-3, nova-2, nova") in a compatibility table |
| 2026-10-01 23:05 ET | Agent | Python SDK type signature | Signature lists current and legacy models side by side with no feature compatibility, so the SDK couldn't resolve the question either | Minor | Expose feature-by-model support in SDK types or docstrings |
| 2026-10-02 | Human | https://developers.deepgram.com/docs/audio-intelligence | Code examples use different models by language: JavaScript and Java use `nova-3`; Python and Go use `nova-2`. An agent copying the Python or Go example would pick an older model. | Minor | Use the current recommended model in every language's example, and check the examples against each other automatically |
| 2026-10-01 23:05 ET | Agent | Docs fetch (via agent's summarizing fetch tool) | Agent reported the *supported* model as the *recommended* one; ambiguous wording gets distorted further when summarized | Observation | Unambiguous, plain statements of defaults and compatibility |
| 2026-10-02 11:19 PM ET | Agent | https://developers.deepgram.com/docs/summarization (Condition B) | The agent looked for the summary at the top level of the `/v1/listen` response, missed `results.summary`, and concluded summarization doesn't work with nova-3, writing that into its README. A controlled test showed the API returned the summary correctly. The page's response example is correct (`summary` inside `results`), but the agent read a condensed version that said only "same level as `channels`" and dropped the example. | Major | State the exact path (`results.summary`) in the prose, not only in the example, since agents often read condensed versions of pages |
| 2026-10-03 8:17 AM ET | Agent | Docs MCP server, https://developers.deepgram.com/_mcp/server (Condition C1) | `searchDocs` failed on every call: `Search failed: {"error":"Failed to fetch from FAI chat service"}`. Still failing at every check through 12:50 PM ET on 2026-10-04, more than 28 hours later. The agent fell back to guessing page URLs, missed the Summarization page, and told the developer summarization exists only on `/v1/read`. | Major | Monitor agent-facing services like production APIs; fall back to static search when the chat service is down |
| 2026-10-03 | Human | https://developers.deepgram.com/developer-tools/agentic-tools vs. the agent-facing Markdown and `llms.txt` versions of the docs | Two different docs MCP servers are advertised. The Markdown and `llms.txt` versions of the docs, which agents read, point AI tools to `https://developers.deepgram.com/_mcp/server`; the Agentic developer tools page lists only `https://api.dx.deepgram.com/kapa/mcp` (alternative `https://deepgram.mcp.kapa.ai`). They come from different providers, and on 2026-10-03 the first one's search failed at every check for more than 28 hours, while the second worked. See [examples](#examples-two-docs-mcp-servers) below. | Major | Advertise one canonical docs MCP server everywhere, or explain when to use each |
| 2026-10-03 | Human | Docs MCP server `https://api.dx.deepgram.com/kapa/mcp` | Requires authentication before any search works. In my setup, a browser sign-in: any Google account worked (no existing Deepgram account needed), with permission to share basic account information; about 22 seconds. The setup page says to add the server "with a single command" and mentions neither the sign-in nor the alternative documented in Deepgram's skills repo (an API key sent as an `Authorization` header). An unattended agent can't complete a browser sign-in. | Minor | State on the setup page that the server requires authentication, and show both options (browser sign-in, or API key header) |
| 2026-10-03 3:02 PM ET | Human | Docs MCP server `https://api.dx.deepgram.com/kapa/mcp` (Condition C-alt setup) | First connection showed "connected" but listed no tools; after a restart, reconnect failed with "Version negotiation probe timed out after 5000ms"; a retry about 3 minutes later succeeded. | Minor | Monitor connection and tool listing, not just availability |
| 2026-10-03 3:10 PM ET | Agent | Docs MCP server `https://api.dx.deepgram.com/kapa/mcp` (Condition C-alt) | The diarization search returned 13 results, mostly GitHub recipes, examples, and discussions. The top result, Deepgram's Python diarization recipe, uses the deprecated `diarize=True`; the Diarization page explaining its replacement never appeared. The agent used the deprecated parameter. | Major | Update recipes when parameters are deprecated; rank current docs pages above recipes for feature questions |

## Human developer walkthrough: Voice Agent tutorial (from memory)
Following https://developers.deepgram.com/docs/build-a-voice-agent-python as a human developer on 2026-09-30, before the agent runs. Recorded from memory; causes weren't investigated unless noted.

| When | Actor | Where | What happened | Severity | Suggested fix |
|---|---|---|---|---|---|
| 2026-09-30 (from memory) | Human | Voice Agent Python tutorial | The code wouldn't run at first because of my local environment setup. The docs site's "Ask AI" feature helped me resolve it. | Minor | Add a short environment checklist (Python version, dependencies) at the top of the tutorial |
| 2026-09-30 (from memory) | Human | Voice Agent Python tutorial | The output WAV files wouldn't play and seemed corrupted. Cause not investigated. | Minor | Describe the expected output files and how to play them |
| 2026-09-30 (from memory) | Human | Voice Agent Python tutorial | With the sample audio, the agent interrupted the speaker and seemed to make assumptions about the conversation. The tutorial didn't say what result to expect or what to explore next, so I couldn't tell whether this was expected. | Major | Add "Expected result" and "What to try next" sections |
| 2026-09-30; rechecked 2026-10-03 | Human | https://github.com/deepgram-devs/deepgram-voice-agent-demo, listed as the "Basic demo" under Implementation examples on https://developers.deepgram.com/docs/build-a-voice-agent | The README says a live demo is at https://deepgram.com/agent, and the repo's About section links there too. The link returned a 404 error on 2026-09-30 and still did on 2026-10-03. | Minor | Fix or remove the link, and check links in example repos automatically |

## Examples: two docs MCP servers
Concrete places where each server is advertised, checked on 2026-10-03.

**1. The agent-facing versions of the docs point to `_mcp/server`.** The Markdown version of each docs page and the `llms.txt` indexes open with a note telling AI clients such as Claude Code and Cursor to connect to the MCP server at
`https://developers.deepgram.com/_mcp/server`. Examples:
- https://developers.deepgram.com/home.md
- https://developers.deepgram.com/home/llms.txt
- https://developers.deepgram.com/docs/stt/getting-started/llms.txt
- https://developers.deepgram.com/docs/flux-tts/overview.md

The regular (HTML) pages carry a different note for AI agents, pointing them to `llms.txt` and the `.md` versions rather than to an MCP server. **The note is in the page's HTML but hidden from view:** a person viewing the page in a browser doesn't see it, but it's in the page source, and it's what an agent reading the HTML page receives. The note begins *"For AI agents: a documentation index is available at the root level at /llms.txt"* and goes on to explain how to get a page-level index or the Markdown version of any page. It appears on, for example:

| HTML page | Points AI agents to | MCP server mentioned in the note? |
|---|---|---|
| https://developers.deepgram.com/docs/summarization | `llms.txt`, `.md` versions | No |
| https://developers.deepgram.com/docs/models-languages-overview | `llms.txt`, `.md` versions | No |
| https://developers.deepgram.com/docs/stt-intelligence-feature-overview | `llms.txt`, `.md` versions | No |
| https://developers.deepgram.com/docs/tts-models | `llms.txt`, `.md` versions | No |
| https://developers.deepgram.com/docs/voice-agent-llm-models | `llms.txt`, `.md` versions | No |
| https://developers.deepgram.com/reference/deepgram-api-overview | `llms.txt`, `.md` versions | No |
| https://developers.deepgram.com/developer-tools/agentic-tools | `llms.txt`, `.md` versions | No (the page body lists the kapa server) |

Some snapshots of the home page and API reference show a variant that also mentions `/llms-full.txt`; none mention an MCP server. *(Checked 2026-10-03: the Summarization page by fetching it directly with a tool, where the note appears, while the same page viewed in a browser shows no such note, though the note is present in the page source; the others through search-index snapshots of the live pages.)*

**The same page, two versions, two different servers.** The Home page shows the contrast most clearly:

| Version | What it tells AI tools |
|---|---|
| HTML (https://developers.deepgram.com) | Its "MCP Server" card links to the Agentic developer tools page, which lists `https://api.dx.deepgram.com/kapa/mcp` |
| Markdown (https://developers.deepgram.com/home.md) | Opens with a note to connect to `https://developers.deepgram.com/_mcp/server` |

A developer reading the site and an agent reading the Markdown version of the same page are pointed to different servers, from different providers, with different authentication requirements.

**2. The server describes itself with its own setup command.** Opening `https://developers.deepgram.com/_mcp/server`
in a browser returns `fern-docs-mcp-server` 1.0.0, one tool (`searchDocs`), and this Claude Code command:
```
claude mcp add --transport http fern_mcp_developers-deepgram-com https://developers.deepgram.com/_mcp/server
```

**3. The Agentic developer tools page lists a different server.** https://developers.deepgram.com/developer-tools/agentic-tools
gives `https://api.dx.deepgram.com/kapa/mcp` as the URL and `https://deepgram.mcp.kapa.ai` as the alternative, with this
Claude Code command:
```
claude mcp add deepgram-docs --scope project --transport http https://api.dx.deepgram.com/kapa/mcp
```
It doesn't mention `_mcp/server`, and it doesn't mention that this server requires a sign-in.

**4. Deepgram's own agent skills have already run into it.** Pull request #11 in Deepgram's public skills repository, *"fix: correct setup-mcp hosted server, document html starter submodules, fix api self-hosted path and regional endpoints"* (https://github.com/deepgram/skills/pull/11). Its description says every claim was verified live on 2026-09-18. The parts relevant here:

- **What the skill said before:** it named `https://api.dx.deepgram.com/kapa/mcp` in six places and presented it as the option for users without an API key.
- **What testing found:**

  | Check (from the PR) | Result reported |
  |---|---|
  | `api.dx.deepgram.com/kapa/mcp`, unauthenticated connection | Rejected: HTTP 401, "Authentication required" (OAuth-protected) |
  | `deepgram.mcp.kapa.ai`, unauthenticated connection | Rejected: HTTP 401, "invalid_token" |
  | Both servers in `claude mcp list` | Both showed `! Needs authentication` (the same status I saw) |
  | `developers.deepgram.com/_mcp/server`, unauthenticated | Worked: HTTP 200, `fern-docs-mcp-server` 1.0.0, one tool, `searchDocs`. The PR notes it wasn't mentioned anywhere in the old skill |

- **What changed:** the skill now presents `_mcp/server` as the credential-free docs server. The two kapa addresses remain, under a heading stating they require authentication.

- **A naming conflict users hit:** the PR adds troubleshooting for an error saying a server named `deepgram-docs` is defined in multiple scopes with different endpoints, which it reproduced and says a user following the old skill instructions would hit.

- **A correction during review:** a reviewer's second, adversarial review (by a separate agent) found that `api.dx.deepgram.com/kapa/mcp` **does** accept a Deepgram API key sent as `Authorization: Token <key>`, returning  HTTP 200. The original test had used an unset environment variable, so it sent an empty credential and read the 401 as "key rejected". Per the review, only `deepgram.mcp.kapa.ai` genuinely rejects the key.

- **Follow-up flagged:** the reviewer noted the repo's README still showed a setup command for the kapa address without
  the authentication header, to be fixed separately.

**What this adds:** this public pull request had identified the authentication requirement and moved the skills to `_mcp/server` about two weeks before this study. On 2026-10-03, the public Agentic developer tools page still listed only the kapa addresses, with no mention of authentication. The review's correction also mirrors finding 3 in this repo: an agent read a failed check as a statement about the product, and an independent check corrected it.

How I found it: a public web search for Deepgram's docs MCP server addresses, while checking which one Deepgram recommends. A later pull request in the same repository ([#16](https://github.com/deepgram/skills/pull/16)) confirms #11 was merged.

**5. On the day of this study, they behaved differently.** `_mcp/server` connected but its search failed on every check from 8:17 AM ET on 2026-10-03 to 12:50 PM ET on 2026-10-04, more than 28 hours. `api.dx.deepgram.com/kapa/mcp` required a sign-in, connected after a
retry, and returned search results around 3:05 PM ET.

