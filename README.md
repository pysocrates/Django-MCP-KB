# Django MCP Knowledge Base

A separate, read-only knowledge base for Django project guidance. Start with
[the index](00-index.md). Its Markdown document format is derived from [MCP Layer Boilerplate](https://github.com/pysocrates/MCP-Layer-Boilerplate), which also provides the Python stdio MCP server used to serve this knowledge base. This repository contains the Django documents; the server lives in the linked repository.
The documents reference Django 6.1 official documentation; verify applicability
against the version used by your project.

## Connect an MCP client

Install the boilerplate server's dependencies first (`uv sync` in its checkout).
Configure a client to launch that server with `MCP_DOCS_DIR` pointing to this
knowledge base. For clients that support `mcpServers` JSON:

```json
{
  "mcpServers": {
    "django-kb": {
      "command": "/absolute/path/to/MCP-Layer-Boilerplate/.venv/bin/python",
      "args": ["/absolute/path/to/MCP-Layer-Boilerplate/server.py"],
      "env": {"MCP_DOCS_DIR": "/absolute/path/to/django-kb"}
    }
  }
}
```

Replace both paths with absolute paths on your machine. For Codex, Claude Code,
Hermes Agent, and Pi, consult the boilerplate README for the respective client
configuration; set `MCP_DOCS_DIR` in the server's environment. Restart or
reconnect after edits, then ask the agent to list or search the Django KB.

To validate on macOS/Linux, from the boilerplate checkout:

```sh
MCP_DOCS_DIR=/absolute/path/to/django-kb .venv/bin/python -m kb_server.validate
```

The server loads numbered Markdown files once at startup. Keep credentials and
private content out of any knowledge base exposed to agents.
