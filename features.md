# Memora Features

Full feature specification, scope boundaries, and open questions.

Related: [README.md](README.md)

---

## Design constraints

Fixed. Every feature below is built within these.

| Constraint | Consequence |
| --- | --- |
| No API keys for users | No paid service is ever called |
| No server, cloud, or hosting | All data in one file on the user's machine |
| No machine learning models | Under 350 MB peak memory, under 5 MB install |
| No network after install | Fully offline, permanently |
| No telemetry | History is never observed by anyone |

---

## v1 scope

### Capture

- [ ] Capture messages from the current conversation, generically, with no site-specific code
- [ ] Capture adapter for ChatGPT, including the full sidebar listing and conversation bodies
- [ ] Capture adapter for Claude
- [ ] Capture adapter for Gemini
- [ ] Watch local conversation files for desktop applications
- [ ] Backfill of previously existing conversations, resumable, with rate limits respected
- [ ] Deduplicate on content hash, detect and record revisions
- [ ] Three fallback tiers per site: network interception, site API, page parsing
- [ ] Graceful degradation: report a broken site in the interface rather than showing a stale index
- [ ] Parser contract tests against recorded real responses, run on every build
- [ ] Per-site health status: last successful capture, success rate, active tier

### Storage

- [ ] Single SQLite database file
- [ ] Conversations, messages, tags, and search index in one file
- [ ] Per-message indexing rather than per-conversation
- [ ] Store source, date, project, model, starred and archived state
- [ ] Full text index using FTS5 with the trigram tokenizer
- [ ] Vocabulary and trigram index built at ingest time
- [ ] Persistent alias table for technical synonyms
- [ ] FTS5 external content tables to avoid duplicated storage
- [ ] Memory mapped access for fast reads
- [ ] Schema versioning and migration

### Search

- [ ] Query normalisation
- [ ] Porter2 stemming
- [ ] Typo correction using trigram distance over the user's own vocabulary
- [ ] Alias expansion into OR groups
- [ ] BM25 ranking via FTS5
- [ ] Term proximity ranking
- [ ] Coverage requirement across multiple distinct concepts
- [ ] Field weighting: title, first prompt, assistant reply, full body
- [ ] Assistant replies weighted above user prompts
- [ ] Relevance feedback expansion, re-ranking the candidate set
- [ ] Optional soft recency boost, never a hard filter
- [ ] Near-duplicate collapsing in results
- [ ] Snippets with highlighted matches
- [ ] Result count and timing reported in the interface
- [ ] Search within a single conversation
- [ ] Search across all sources and a single source
- [ ] Saved searches with filters preserved
- [ ] Recent search history

### Temporal search

- [ ] Absolute date range
- [ ] Relative phrases such as "last week" or "yesterday"
- [ ] Approximate dates, treated as a widened range with distance-based ranking
- [ ] Time of day and day of week filters
- [ ] Sort by relevance, recency, or length

### Organisation

- [ ] Automatic topic grouping via TF-IDF and NMF
- [ ] Topics auto-named from their highest-weighted terms
- [ ] Rename, merge, and split topics
- [ ] Classification by example: label a few, sort the rest
- [ ] Manual tags on conversations
- [ ] Coarse category buckets: tech, study, work, personal, health, and configurable additions
- [ ] Clustering of unlabelled conversations
- [ ] Promote any cluster to a named collection
- [ ] Saved collections and pins
- [ ] Timeline view of activity
- [ ] Conversation length and volume statistics

### Interface

- [ ] Command palette opened by keyboard shortcut, available anywhere
- [ ] Live capture indicator showing per-site status
- [ ] Result list with matching passage shown in context
- [ ] Click a result to open the original conversation in the original application
- [ ] Full thread viewer for conversations with no live source
- [ ] Browse mode by topic, tag, and date
- [ ] Topic and tag browser
- [ ] One-click export of a conversation to Markdown or plain text
- [ ] Setup and capture status screen
- [ ] Keyboard-first navigation throughout
- [ ] Light and dark themes

### Notable differences between sources

Handled by adapters, listed here because they are the source of most capture bugs.

| Source | Difference |
| --- | --- |
| ChatGPT | Message history is a branching tree, not a list. The visible thread is one path through it. |
| Claude | Same branching, plus pagination on the conversation list is broken and must be worked around. A "simple" fetch mode silently returns empty tool results. |
| Gemini | Requests use an undocumented batch RPC. Capture may be limited to the current conversation. |

### Explicitly out of scope for v1

- Prose answers to questions about history. Replaced by a structured evidence timeline, which is free and more trustworthy.
- Neural or embedding search
- Cloud sync, accounts, sharing, teams
- Mobile application support
- Any hosted API
- Telemetry, analytics, crash reporting
- A polished installer

---

## v2 and beyond

- [ ] Neural embeddings as an optional, additive tier that reranks lexical results. The lexical index is not replaced, so this stays an upgrade rather than a migration.
- [ ] One-line summaries per conversation
- [ ] Cross-device sync via manual export and import
- [ ] Additional platforms: Grok, Perplexity, Copilot, AI Studio, LM Studio, Claude Code
- [ ] MCP server, allowing any agent to search the history as a tool
- [ ] Continue a conversation in the original application from a result
- [ ] Read-only timeline of a topic across time
- [ ] Export the full history as JSON, Markdown, or plain text

---

## Use cases

- "Find that Python bug fix from last week"
- "What was the SQL query we discussed?"
- "Show all chats tagged #research"
- "Find the conversation where we chose Postgres over Mongo"
- "What was that error about the CORS header?"
- "Show me everything about my placement preparation"
- "Which chats did I have while working on the college project?"
- "Find the conversation where I asked about useEffect cleanup"

---

## Success metrics

- Time to find a conversation under 3 seconds
- Works for 100 or more conversations with no noticeable slowdown
- Zero privacy complaints
- Capture failure is always visible to the user, never silent

---

## Open questions

- Which platforms first: ChatGPT only, or ChatGPT and Claude together?
- When to backfill: on first run, on demand, or continuously in the background?
- Search entry point: a floating button, a keyboard shortcut, or both?
- Folder structure or search only, or a flat topic model with manual collections on top?
- Should the local service start with the operating system, or on demand?
- How much of the conversation list to fetch eagerly versus lazily on open?
- Should assistant replies be searchable by default, or opt-in?

---

## Resources

- Chrome extension development: https://developer.chrome.com/docs/extensions/
- Manifest V3 migration guide: https://developer.chrome.com/docs/extensions/develop/migrate
- SQLite FTS5: https://sqlite.org/fts5.html
- SQLite trigram tokenizer: https://sqlite.org/fts5.html#the_trigram_tokenizer
- Porter2 stemmer: https://snowballstem.org/algorithms/english/stemmer.html
- BM25: https://en.wikipedia.org/wiki/Okapi_BM25
- Non-negative matrix factorisation: https://en.wikipedia.org/wiki/Non-negative_matrix_factorization
- HDBSCAN: https://hdbscan.readthedocs.io/
- Model Context Protocol: https://modelcontextprotocol.io/
- OpenAI conversation endpoints, observed in browser network traffic
- Claude conversation endpoints, observed in browser network traffic
