# Skill MCP Services

Two stdio MCP services for ModelScope custom MCP deployment:

- mineru-book-mcp: bounded upload, SHA-256 verified download, MinerU Agent/Standard parsing, local fallback, and book-to-skill workflow access.
- fireworks-graph-mcp: bounded upload, geometry checked SVG rendering, PNG export, offline HTML export, and verified download.

Both are installed with uvx from release wheels. The ModelScope JSON templates
use the public GitHub release paths. Set MINERU_TOKEN only in the ModelScope
environment when Standard API is needed. No token is included here. Each service
runs in its own isolated uvx environment.

The document service does not call an LLM: the host model reads workflow
instructions, drafts structured Skill Markdown, then calls package_skill.

## ModelScope MCP configurations

### MinerU and book-to-skill

```json
{
  "mcpServers": {
    "mineru": {
      "command": "uvx",
      "args": [
        "--from",
        "https://raw.githubusercontent.com/TerryHank/skill-mcp-services/main/releases/mineru_book_mcp-0.1.0-py3-none-any.whl",
        "mineru-book-mcp"
      ],
      "env": { "SKILL_SERVICE_DATA": "/tmp/mineru-book-mcp-data" }
    }
  }
}
```

### Fireworks technical graph

```json
{
  "mcpServers": {
    "fireworks": {
      "command": "uvx",
      "args": [
        "--from",
        "https://raw.githubusercontent.com/TerryHank/skill-mcp-services/main/releases/fireworks_graph_mcp-0.1.0-py3-none-any.whl",
        "fireworks-graph-mcp"
      ],
      "env": { "SKILL_SERVICE_DATA": "/tmp/fireworks-graph-mcp-data" }
    }
  }
}
```

In ModelScope choose custom MCP, select `uvx`, and paste one configuration at
a time. The wheel URLs are public and contain no credentials. Add `MINERU_TOKEN`
only as a protected deployment environment variable when Standard API features
are needed.
