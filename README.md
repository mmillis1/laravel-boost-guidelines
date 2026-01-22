# Boost Guidelines

Reusable development guidelines and MCP configurations for AI-assisted development.

## Setup

### Symlink Guidelines

Symlink individual markdown files to your project's `.ai/guidelines/` directory:

````bash
# Symlink a specific guideline
ln -s /path/to/boost-guidelines/guidelines/laravel.md /path/to/project/.ai/guidelines/laravel.md

### Copy MCP Files

Copy MCP configuration files to your project root (copies, not symlinks):

```bash
# Copy mcp.json as .mcp.json
cp /path/to/boost-guidelines/mcp/mcp.json /path/to/project/.mcp.json

# Copy opencode.json as opencode.json
cp /path/to/boost-guidelines/mcp/opencode.json /path/to/project/opencode.json
````
