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
