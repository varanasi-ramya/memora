# Memora

A local search engine for your AI conversation history.

Memora reads your conversations with ChatGPT, Claude, and Gemini while you have them, stores them in a single file on your own machine, and gives you one search box that looks across all of them at once.

It is free to build, free to run, and free for users. It requires no API keys, no account, and no network connection after installation. It runs no machine learning models.

---

## The problem

Your AI conversations are scattered across separate websites, each with its own search box that only searches that one site. ChatGPT can find ChatGPT chats. Nothing finds the conversation you had with Claude three months ago.

Even within a single site, search is literal. It finds a chat when you type the exact words that are in it. It cannot find "the conversation about the database problem" unless you remember the exact phrasing you used at the time.

---

## What Memora does

1. **Captures** every message you send and receive, quietly, while you chat normally.
2. **Stores** everything in one local database file, turn by turn.
3. **Searches** using word matching, typo correction, synonym expansion, and topic modelling, all computed locally.
4. **Organises** conversations automatically into topics you can browse and refine.

---

## Design constraints

These are fixed and not negotiable. Any feature that violates them does not ship.

| Constraint | Consequence |
| --- | --- |
| No API keys for users | Nothing calls a paid service, ever |
| No server, cloud, or hosting | All data lives in one file on the user's machine |
| No machine learning models | Peak memory under 350 MB, install under 5 MB |
| No network calls after install | Works fully offline, forever |
| No telemetry or analytics | The user's history is never observed by anyone |

The no-model constraint is a deliberate architectural choice, not a limitation to be removed later. See [Why no models](#why-no-models).

---

## How it works

Memora has five parts.

### 1. Capture (the extension)

A browser extension runs inside the site. It wraps the network request the site already makes to load a conversation, lets the real data pass through untouched, and keeps a copy.

The script executes in the page's own security context, so cookies and session state are handled by the browser. Memora never reads, stores, or transmits a credential.

This approach is generic and works on any site without site-specific code. Site-specific adapters are only needed to reach conversations you have not visited yet.

For desktop applications with no browser interface, Memora watches their local conversation files on disk instead.

### 2. Storage (one file)

A single SQLite database on the user's machine holds everything: conversations, messages, tags, and the search index.

Messages are indexed individually rather than as whole conversations. This means a search hit jumps directly to the matching message instead of the start of a long thread, and it keeps long conversations within a workable size.

### 3. Search

A deterministic pipeline, run in order. Every stage is inspectable, which means a wrong result can be traced to the exact stage that caused it.

```
query
  -> normalise        lowercase, strip punctuation
  -> stem             "running" -> "run"
  -> correct typos    trigram distance over the user's own vocabulary
  -> expand aliases   postgres | postgresql | psql | relational
  -> BM25             full text ranking, top 200 candidates
  -> proximity        query terms near each other rank higher
  -> coverage         require more than one distinct concept to match
  -> field weighting  title > first prompt > assistant reply
  -> feedback         expand from the top results, re-query
  -> recency          soft boost, never a hard filter
  -> diversity        collapse near-duplicates
  -> final 10
```

Assistant replies are weighted more heavily than user prompts, because the answer to a problem is usually written by the model, not asked for by the user.

### 4. Organisation

Conversations are grouped by topic using non-negative matrix factorisation over a term frequency matrix. Topics are named automatically by their highest-weighted terms, and can be renamed, merged, or split.

A conversation can also be classified by example: select a handful of conversations, label them, and the rest are sorted by similarity to those examples.

Unlabelled conversations are grouped into clusters, which surfaces topics the user never organised.

### 5. Interface

A command palette opened with a keyboard shortcut, available from anywhere. Results show the matching passage in context, and a result opens the original conversation in the original application at the right place.

---

## Why no models

Neural embedding models cost roughly 130 MB, require a download on first run, complicate packaging inside a Manifest V3 extension, and add inference scheduling to the query path.

They buy exactly one capability: tolerance of paraphrase. Everything else in this product is unaffected by their absence.

Personal conversation history is the best possible case for lexical methods. It is a single archive, in one language, with a small repeated personal vocabulary. A term frequency model built from that archive is a better prior than a general-purpose model, because it knows which words that user actually uses.

This choice also removes the single largest source of ongoing fragility. Nothing about this architecture breaks when a model, a runtime, or a hosting provider changes.

### Known limitations

These are the cases where lexical search is genuinely weaker, stated plainly rather than hidden.

- Describing a conversation in words that never appear anywhere in it
- Searching in one language for conversations written in another
- Queries where the useful term is the outcome, not the topic, such as "the fix that finally worked"

The third is partially mitigated by weighting assistant replies. The first two are the reason the neural tier exists as a later, optional addition rather than a core dependency.

---

## Tech stack

Everything is open source. There are no paid dependencies.

| Layer | Choice |
| --- | --- |
| Extension | TypeScript, Manifest V3, esbuild |
| Local service | Node.js, Fastify, bound to localhost |
| Database | SQLite with FTS5, trigram tokenizer |
| Full text search | BM25 via FTS5 |
| Topic modelling | TF-IDF plus NMF |
| Clustering | HDBSCAN or k-means |
| Similarity | TF-IDF cosine and truncated SVD (LSI) |
| Text processing | Porter2 stemmer, trigram edit distance, curated alias table |
| Interface | React, Vite, Tailwind |
| Agent access | MCP server over stdio |

TypeScript is used end to end so the extension, the service, and the interface share types.

---

## Privacy

- All conversation data is stored in a single file on the user's own machine.
- Nothing is transmitted anywhere, at any point.
- No account, no sign-in, no telemetry, no crash reporting.
- The only network access the extension makes is to the sites the user is already logged into, using the user's existing session.

Users can verify this. The network panel shows every request the extension makes.

---

## Installation

The intended distribution is a browser extension plus a local service.

1. Install the extension for Chrome, Edge, or Firefox.
2. Install the local service, which runs on the user's machine and holds the database.
3. Open the command palette.

There is no account to create and no server to configure.

---

## Project layout

```
memora/
  extension/     browser extension, capture layer
  service/       local Node service, database, search pipeline
  ui/            React command palette and thread viewer
  shared/        shared types and parsing adapters
  data/          the user's database, not in version control
```

---

## Status

Early development. See [features.md](features.md) for the full feature specification, what is out of scope for v1, and the open design questions.

---

## Contributing

Issues and pull requests are welcome. The most valuable contributions are capture adapters for platforms not yet supported, and entries for the alias table, which directly improves search quality for technical vocabulary.

---

## License

MIT
