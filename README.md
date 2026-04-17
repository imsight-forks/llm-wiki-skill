# llm-wiki

**An OpenClaw / Codex Agent Skill for building Karpathy-style LLM knowledge bases.**

> Experimental skill - will iterate over time.
> Please send feedback in GitHub issues.

Inspired by [Andrej Karpathy's llm-wiki Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) and the community's work building on it.

## What This Is

Instead of RAG, where raw documents are re-retrieved on every query, this pattern has the LLM compile raw sources into a persistent, cross-linked Markdown wiki. Every `compile`, `ingest`, `query`, `lint`, and `audit` pass makes the wiki richer.

- You own sourcing raw material, asking questions, steering direction, and filing feedback.
- The LLM owns writing, cross-referencing, filing, bookkeeping, and applying feedback.

The repository now has two self-contained deliverables:

- **`llm-wiki-all-in-one/`** - the agent skill plus a deployable local web viewer.
- **`plugins/obsidian-audit/`** - an Obsidian plugin that writes anchored feedback files to `audit/`.

The old top-level standalone `llm-wiki/`, `web/`, and `audit-shared/` packages have been folded into the all-in-one skill or the plugin.

## Install The Skill

Copy or symlink the all-in-one skill into your agent's skills directory.

```bash
# Claude-style skill directory
cp -r llm-wiki-all-in-one ~/.claude/skills/llm-wiki-all-in-one

# Codex-style skill directory
cp -r llm-wiki-all-in-one ~/.codex/skills/llm-wiki-all-in-one
```

Inside this repository, the project-scope Codex skill can be a symlink:

```bash
ln -s ../../extern/llm-wiki-skill/llm-wiki-all-in-one .codex/skills/llm-wiki-all-in-one
```

## Quick Start

```bash
# 1. Scaffold a new wiki
python3 llm-wiki-all-in-one/scripts/scaffold.py ~/my-wiki "My Research Topic"

# 2. Add a source
cp my-article.md ~/my-wiki/raw/articles/

# 3. Tell your agent: "ingest raw/articles/my-article.md"

# 4. Ask questions: "what does the wiki say about X?"

# 5. Run lint periodically
python3 llm-wiki-all-in-one/scripts/lint_wiki.py ~/my-wiki

# 6. File a comment from the web viewer or Obsidian plugin, then process it
python3 llm-wiki-all-in-one/scripts/audit_review.py ~/my-wiki --open
# then tell the agent: "audit: process the open comments"
```

## Repository Contents

```text
llm-wiki-skill/
├── llm-wiki-all-in-one/
│   ├── SKILL.md
│   ├── references/
│   ├── scripts/
│   │   ├── scaffold.py
│   │   ├── lint_wiki.py
│   │   ├── audit_review.py
│   │   └── deploy_viewer.py
│   ├── subskills/
│   │   └── viewer-deploy.md
│   └── viewer/
│       ├── audit-shared/
│       └── web/
└── plugins/
    └── obsidian-audit/
        ├── audit-shared/
        ├── src/
        └── scripts/
```

`llm-wiki-all-in-one/viewer/audit-shared/` and `plugins/obsidian-audit/audit-shared/` are intentionally duplicated so each deliverable can be installed and built on its own.

## Deploy The Web Viewer

The local web viewer renders the wiki with Mermaid, KaTeX, and wikilinks. It can file feedback from selected browser text into the wiki's `audit/` directory.

Use the deploy helper from the all-in-one skill:

```bash
python3 llm-wiki-all-in-one/scripts/deploy_viewer.py \
  --install-dir ~/llm-wiki-viewer \
  --wiki "/path/to/your/wiki-root" \
  --port 8080
```

The helper copies the packaged viewer source into the installation directory, installs dependencies with npm when available or bun as a fallback, builds the viewer, excludes generated files from local Git tracking, and launches on `127.0.0.1`.

For setup without launching:

```bash
python3 llm-wiki-all-in-one/scripts/deploy_viewer.py \
  --install-dir ~/llm-wiki-viewer \
  --wiki "/path/to/your/wiki-root" \
  --launch-mode no-launch
```

Then open the reported local URL, usually `http://127.0.0.1:8080`.

## Build The Obsidian Plugin

The plugin is self-contained and depends on its local `audit-shared/` copy.

```bash
cd plugins/obsidian-audit
npm install
npm run build
npm run link -- "/path/to/your/Obsidian vault"
```

Enable **LLM Wiki Audit** in Obsidian settings after linking.

## Audit Shared Maintenance

The web viewer and Obsidian plugin use the same audit schema, anchor algorithm, ID generator, and serializer. Because the two deliverables are self-contained, `audit-shared` exists in two places:

```text
llm-wiki-all-in-one/viewer/audit-shared/
plugins/obsidian-audit/audit-shared/
```

When audit schema code changes, refresh the plugin copy from the all-in-one copy and rebuild the plugin.

## Use Cases

- Research deep-dives over papers, articles, and source code.
- Personal or team knowledge bases that improve over time.
- Reading companions that turn notes and chapters into an explorable wiki.
- Feedback-driven AI-written documentation, where corrections live in `audit/`.

## Related Work

- [Karpathy's original Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [pedronauck/skills karpathy-kb](https://github.com/pedronauck/skills/tree/main/skills/karpathy-kb)
- [Astro-Han/karpathy-llm-wiki](https://github.com/Astro-Han/karpathy-llm-wiki)
- [qmd](https://github.com/tobi/qmd)

## License

MIT
