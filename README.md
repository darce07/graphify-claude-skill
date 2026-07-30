<div align="center">

# 🕸️ Graphify

### Turn any codebase, doc pile, or media folder into a queryable knowledge graph

*Drop in a folder — code, docs, papers, images, video — and get back a persistent, honest knowledge graph you can query in plain language, trace shortest paths across, and hand to other agents over MCP.*

![Version](https://img.shields.io/badge/version-0.9.26-blue)
![Platform](https://img.shields.io/badge/platform-Claude%20Code-8A63D2)
![Status](https://img.shields.io/badge/status-active-success)

[What is it?](#-what-is-this) ·
[Installation](#-installation) ·
[How it activates](#-how-it-activates) ·
[What it does](#-what-it-does) ·
[Structure](#-repository-structure) ·
[Author](#-author)

</div>

---

## 📖 What is this?

**Graphify** is a Claude Code skill that turns `/graphify <path>` into a full pipeline: scan a folder, extract entities and relationships from every file it understands, build a graph, detect communities inside it, and hand back three artifacts — an interactive HTML visualization, a GraphRAG-ready `graph.json`, and a plain-language `GRAPH_REPORT.md`.

It works on mixed corpora, not just source code. Code is parsed structurally with AST (no LLM, no API key, free). Docs, papers, and images go through semantic extraction — either parallel Claude subagents or a Gemini backend if you have a key set. Video and audio get transcribed with Whisper first and folded in as documents. Every extracted relationship is tagged `EXTRACTED` (explicit in the source), `INFERRED` (a reasonable inference), or `AMBIGUOUS` (flagged, never silently dropped) — so the graph never claims more certainty than it has.

Once a graph exists at `graphify-out/graph.json`, graphify's real value kicks in: any natural-language question about the codebase is answered by querying the existing graph instead of re-extracting from scratch. That's the fast path — the skill is written to prefer it over rebuilding.

## ✨ Key features

| Feature | What it does |
|---|---|
| **Structural + semantic extraction** | Code parsed via AST (free, no LLM); docs/papers/images extracted by parallel subagents or Gemini |
| **Honest audit trail** | Every edge is `EXTRACTED`, `INFERRED`, or `AMBIGUOUS` with a confidence score — never invented |
| **Community detection** | Clusters the graph and names each cluster in plain language (e.g. "Attention Mechanism", "Training Pipeline") |
| **God nodes & surprising connections** | Surfaces the most-connected nodes and cross-community bridges you wouldn't think to ask about |
| **`query` / `path` / `explain`** | BFS/DFS traversal against the existing graph, shortest path between two concepts, plain-language node explanations |
| **Vocabulary-aware query expansion** | Expands your question against the graph's own label vocabulary before searching, so wording/language mismatches don't collapse results to noise |
| **Incremental `--update`** | Re-extracts only new or changed files instead of rebuilding the whole graph |
| **`--watch` mode** | Watches a folder and auto-rebuilds on code changes with no LLM calls; flags doc/image changes for a manual update |
| **`add <url>`** | Ingests a URL (YouTube, Twitter/X, arXiv, PDF, image, or any webpage) straight into the corpus |
| **Video/audio transcription** | Transcribes video and audio with Whisper, using a domain-aware prompt derived from the graph's own god nodes |
| **Cross-repo & monorepo merge** | Clones and builds multiple GitHub repos or subfolders separately, then merges them into one graph with `repo` attributes |
| **Multiple export targets** | HTML, SVG, GraphML, Obsidian vault, agent-crawlable wiki, Neo4j, FalkorDB, and a stdio MCP server for other agents |
| **Git hook & CLAUDE.md integration** | `graphify hook install` rebuilds after every commit; `graphify claude install` makes the graph always-on for future sessions |
| **Self-improving query loop** | Saves each query/path/explain answer back into the graph with an outcome (`useful` / `dead_end` / `corrected`) so later sessions learn from it |
| **Graph health check** | Read-only diagnostic for dangling edges, missing endpoints, self-loops, and collapsed edges before labeling |

## 🚀 Installation

### Global install (recommended)

```bash
# Windows
cd C:\Users\<your-user>\.claude\skills
git clone https://github.com/darce07/graphify-claude-skill.git graphify

# macOS / Linux
cd ~/.claude/skills
git clone https://github.com/darce07/graphify-claude-skill.git graphify
```

### Per-project install

```bash
cd /path/to/your/project
mkdir -p .claude/skills
cd .claude/skills
git clone https://github.com/darce07/graphify-claude-skill.git graphify
```

Restart Claude Code and the skill will appear automatically. A `.graphify_version` file in the skill folder tracks which version of the skill is installed.

The skill itself installs the `graphify` Python package on first run (via `uv tool install`, `pipx`, or `pip`, auto-detected) — no manual setup step required beyond cloning this repo.

## 🎯 How it activates

Graphify triggers in two ways:

- **Explicitly**, via `/graphify` and its subcommands — `/graphify <path>`, `/graphify <github-url>`, `/graphify query "<question>"`, `/graphify path "A" "B"`, `/graphify explain "<node>"`, `/graphify add <url>`, plus flags like `--update`, `--watch`, `--mode deep`, `--directed`, `--obsidian`, `--wiki`, `--neo4j`, `--mcp`.
- **Automatically**, whenever the user asks a question about a codebase, its architecture, or file relationships — especially when `graphify-out/graph.json` already exists. In that case the skill skips straight to running `graphify query` against the existing graph instead of rebuilding it.

No API key is required to activate it. Code-only corpora need nothing at all; semantic extraction for docs/papers/images uses Gemini only if `GEMINI_API_KEY`/`GOOGLE_API_KEY` is already set, and otherwise falls back to the host agent (e.g. Claude Code subagents) as the extractor.

## 🏗️ Repository structure

```
graphify/
├── SKILL.md                     # The skill definition: usage, full pipeline (Steps 0-9), honesty rules
├── .graphify_version             # Tracks the installed skill version
└── references/                   # Loaded on demand, only when the relevant flag/flow is used
    ├── add-watch.md              # /graphify add <url> and --watch (folder monitoring)
    ├── exports.md                 # --wiki, --neo4j, --falkordb, --svg, --graphml, --mcp, benchmark
    ├── extraction-spec.md         # Exact JSON schema + prompt given to semantic extraction subagents
    ├── github-and-merge.md        # Cloning GitHub repos, cross-repo merge, monorepo/multi-subfolder merge
    ├── hooks.md                   # Post-commit hook install and native CLAUDE.md integration
    ├── query.md                   # /graphify query, path, explain — traversal, vocab expansion, save-result
    ├── transcribe.md              # Video/audio transcription via Whisper before extraction
    └── update.md                  # --update (incremental re-extraction) and --cluster-only
```

## 👤 Author

**darce07**
- GitHub: [@darce07](https://github.com/darce07)

---

<div align="center">

*The graph is the map. Graphify's job is to be the guide.*

</div>
