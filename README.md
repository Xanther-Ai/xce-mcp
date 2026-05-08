# Xanther Context Engine (XCE) — MCP Server

<p align="center">
  <img src="assets/xce-hero.png" alt="Xanther Context Engine" width="700" />
</p>

<p align="center">
  <strong>Give your coding agent deep codebase understanding via MCP.</strong>
</p>

<p align="center">
  <a href="https://xanther.ai">Website</a> •
  <a href="https://app.xanther.ai">Dashboard</a> •
  <a href="https://discord.gg/YaBekKpR">Discord</a> •
  <a href="https://www.npmjs.com/package/xanther-cli">npm</a> •
  <a href="https://x.com/xantherai">Twitter</a>
</p>

---

## What is XCE?

Xanther Context Engine (XCE) is an MCP server that gives your coding agent precise architectural context on every tool call. Instead of your agent guessing where things are in a codebase, XCE provides:

- **Architecture Context** — HLD, LLD, component descriptions for any file or symbol
- **Semantic Search** — find code by meaning, not just text matching
- **Traceability** — trace from a function up to its module-level architecture
- **Impact Analysis** — understand what's affected when files change

Powered by the **PRAT** (Persistent Recursive Abstract Tree) algorithm.

## Benchmark Results

All results on [SWE-bench Verified](https://www.swebench.com/) (500 instances) using mini-swe-agent:

| Setup | Resolve Rate | Cost/Instance |
|---|---|---|
| **MiniMax M2.5 + XCE** | **78.2%** | $0.22 |
| Claude 4.5 Opus (baseline) | 76.8% | $0.75 |
| MiniMax M2.5 (baseline) | 75.8% | $0.07 |
| **Sonnet 4.0 + XCE** | **73.4%** | $0.22 |
| Sonnet 4.0 (baseline) | 66.0% | $0.22 |

78.2% is a direct result (not oracle). Full data: [github.com/Xanther-Ai/xce-benchmarks](https://github.com/Xanther-Ai/xce-benchmarks)

## Quick Start

### 1. Get your API key

Sign up at [app.xanther.ai](https://app.xanther.ai) and generate an API key from the dashboard.

### 2. Index your repository

```bash
npx xanther-cli init --api-key xce_your_key_here
```

The CLI will:
- Check if your repo is already indexed (community repos are pre-indexed)
- Submit your repo for indexing if needed
- Output the MCP config with your `repo_id` embedded in the URL
- Install a git post-commit hook for auto-sync
- Generate steering files for your IDE

### 3. Add MCP config to your IDE

The CLI outputs this after indexing. Copy it into your IDE's MCP config file:

```json
{
  "mcpServers": {
    "xanther-xce": {
      "url": "https://mcp.xanther.ai/sse?repo_id=YOUR_REPO_ID",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

**The `repo_id` parameter** tells XCE which indexed codebase to serve context from. It's embedded in the URL so your agent automatically targets the right repo without any extra configuration.

| IDE | Config File Location |
|---|---|
| Kiro | `.kiro/settings/mcp.json` |
| Claude Code | `~/.claude/mcp.json` |
| Cursor | `.cursor/mcp.json` |
| Windsurf | `.windsurf/mcp.json` |
| OpenCode | `~/.config/opencode/config.json` |
| Cline | `.vscode/mcp.json` |

### 4. Add steering rules (recommended)

Steering files tell your agent to **always** use XCE for context queries instead of reading files blindly. Without steering, the agent may fall back to manual file exploration. With steering, it consistently uses XCE first — resulting in ~20% fewer tokens and better results.

The CLI generates these automatically during `xanther-cli init`. You can also create them manually:

#### Kiro — `.kiro/steering/xce.md`

```markdown
---
inclusion: auto
---
Always use the xanther-xce MCP tools for codebase understanding before reading files directly.

- Call `xce_get_context` as your FIRST step when starting any task. It combines search, architecture context, and tracing into one call.
- Use `xce_architecture_context` before modifying any file to understand its role, dependencies, and what calls it.
- Use `xce_impact_analysis` before making changes that affect multiple files to understand downstream effects.
- Use `xce_search` to find code by meaning when you need to locate specific functionality.
- Use `xce_trace` to understand how a function connects to the broader architecture.

Prefer XCE context over grep, find, or reading files for understanding code structure. XCE provides architectural context that file reading alone cannot.
```

#### Claude Code — `CLAUDE.md` (project root)

```markdown
## Codebase Context

This project uses Xanther Context Engine (XCE) via MCP for codebase understanding.

Always use xanther-xce MCP tools before reading files:
- Call `xce_get_context` first on any task — it returns architecture, code locations, and relationships
- Use `xce_architecture_context` before modifying any file
- Use `xce_impact_analysis` before multi-file changes
- Use `xce_search` to find code by meaning instead of grep
- Use `xce_trace` to understand how code connects to the broader architecture

XCE provides HLD (high-level design), LLD (low-level design), call graphs, and component descriptions. This is faster and more accurate than reading files individually.
```

#### Cursor — `.cursorrules` (project root)

```
Always use xanther-xce MCP tools for codebase understanding before reading files directly.

- Call xce_get_context as your first action on any task
- Use xce_architecture_context before modifying any file to understand its role and dependencies
- Use xce_impact_analysis before making changes that affect multiple files
- Use xce_search to find code by meaning instead of grep/find
- Use xce_trace to understand how a function connects to the broader architecture

Prefer XCE context over file reading for understanding code structure. XCE provides architectural context (HLD, LLD, call graphs) that file reading alone cannot.
```

#### Windsurf — `.windsurfrules` (project root)

```
Always use xanther-xce MCP tools for codebase understanding before reading files directly.

- Call xce_get_context as your first action on any task
- Use xce_architecture_context before modifying any file
- Use xce_impact_analysis before multi-file changes
- Use xce_search to find code by meaning
- Use xce_trace to understand architectural relationships

XCE provides HLD, LLD, call graphs, and component descriptions — faster and more accurate than reading files individually.
```

#### Cline — `.clinerules` (project root)

```
Always use xanther-xce MCP tools for codebase understanding before reading files directly.

- Call xce_get_context as your first action on any task
- Use xce_architecture_context before modifying any file to understand its role and dependencies
- Use xce_impact_analysis before making changes that affect multiple files
- Use xce_search to find code by meaning instead of grep/find

XCE provides architectural context (HLD, LLD, call graphs, component descriptions) that is faster and more accurate than reading files individually.
```

#### OpenCode — Add to system prompt or project config

```
Always use xanther-xce MCP tools for codebase understanding before reading files.
Call xce_get_context first on any task. Use xce_architecture_context before modifying code.
Use xce_impact_analysis before multi-file changes. Use xce_search instead of grep.
```

## Available Tools

Your agent gets access to these five tools via MCP:

### `xce_get_context`

**The most powerful tool.** Give it a natural language description of what you're working on, and it returns full architectural context — relevant code, design docs, call graphs, and related components.

```
Input:  "Fix the race condition in FileBasedCache concurrent writes"
Output: HLD module info, LLD component details, call graph, related files, code snippets
```

Use this as your **first tool call** when starting any task.

### `xce_search`

Semantic search across your indexed codebase. Finds code by meaning, not just text.

```
Input:  "authentication middleware that validates JWT tokens"
Output: Matching functions, classes, and their architectural context
```

### `xce_architecture_context`

Get the full architectural context for any file or symbol — what it does, how it connects to other parts, its HLD module and LLD component.

```
Input:  "django/middleware/csrf.py"
Output: Component description, callers, dependencies, design role
```

### `xce_trace`

Trace relationships from code to design artifacts. Understand how a specific function connects to the broader architecture.

```
Input:  source="validate_token", target_level="hld"
Output: Function → class → module → architectural role
```

### `xce_impact_analysis`

Analyze the impact of changing specific files. Know what breaks before you break it.

```
Input:  changed_files=["src/auth/middleware.py", "src/auth/tokens.py"]
Output: Affected modules, downstream dependencies, risk assessment
```

## Community Repos (Free)

These repos are pre-indexed and available to all users with any API key:

| Repository | repo_id | Nodes | Language |
|---|---|---|---|
| django/django | `django-django` | 14,520 | Python |
| lobehub/lobe-chat | `community--lobe-chat` | 92,139 | TypeScript |
| sympy/sympy | `community--sympy` | 8,450 | Python |
| scikit-learn/scikit-learn | `community--scikit-learn` | 6,200 | Python |
| matplotlib/matplotlib | `community--matplotlib` | 5,800 | Python |
| sphinx-doc/sphinx | `community--sphinx` | 4,100 | Python |
| pytest-dev/pytest | `community--pytest` | 3,200 | Python |
| pydata/xarray | `community--xarray` | 2,800 | Python |
| astropy/astropy | `community--astropy` | 3,500 | Python |
| pallets/flask | `community--flask` | 3,280 | Python |

To query a community repo, use its `repo_id` in the URL:

```json
{
  "mcpServers": {
    "xanther-xce": {
      "url": "https://mcp.xanther.ai/sse?repo_id=django-django",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

## How It Works

```
┌─────────────────┐     MCP/SSE    ┌─────────────────┐
│  Coding Agent   │◄──────────────►│   XCE Server    │
│  (Claude Code,  │                │                 │
│   Kiro, Cursor, │                │  PRAT Algorithm │
│   Windsurf,     │                │  + Structured   │
│   OpenCode,     │                │    Index        │
│   Cline)        │                │                 │
└─────────────────┘                └─────────────────┘
         │                                  │
         │ repo_id in URL                   │ Indexed codebase
         │ API key in header                │ (architecture + code + embeddings)
         ▼                                  ▼
   Agent gets precise              Your repo indexed via
   architectural context           xanther-cli or dashboard
```

1. **Index** — `xanther-cli init` indexes your repo into a structured codebase index
2. **Connect** — Add the MCP config (with repo_id) to your coding agent
3. **Query** — Your agent calls XCE tools when it needs codebase context
4. **Context** — XCE returns precise architectural context, not just raw code

## Supported Languages

| Language | Parser | Status |
|---|---|---|
| Python | AST (built-in) | Stable |
| TypeScript | tree-sitter | Stable |
| JavaScript | tree-sitter | Stable |
| TSX/JSX | tree-sitter | Stable |

More languages coming soon. Request in [Discord](https://discord.gg/YaBekKpR).

## Pricing

| Plan | Queries/month | Repos | Price |
|---|---|---|---|
| Free | 100 | 3 | $0 |
| Starter | 2,000 | 5 | $8/mo |
| Pro | 10,000 | 20 | $15/mo |
| Unlimited | Unlimited | Unlimited | $20/mo |

[View pricing](https://xanther.ai/pricing)

## Links

- [Xanther Website](https://xanther.ai)
- [Dashboard](https://app.xanther.ai)
- [Xanther CLI (npm)](https://www.npmjs.com/package/xanther-cli)
- [Xanther CLI (GitHub)](https://github.com/Xanther-Ai/xanther-cli)
- [Discord Community](https://discord.gg/YaBekKpR)
- [Twitter](https://x.com/xantherai)
- [Benchmark Results](https://xanther.ai/benchmarks)
- [Benchmark Data (GitHub)](https://github.com/Xanther-Ai/xce-benchmarks)

## License

MIT — see [LICENSE](LICENSE) for details.
