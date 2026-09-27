# Memora Features

Full feature specification for v1 and beyond.

Related: [README.md](README.md)

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

## Success metrics

- Time to find a conversation under 3 seconds
- Works for 100 or more conversations with no noticeable slowdown
- Zero privacy complaints
- Capture failure is always visible to the user, never silent
