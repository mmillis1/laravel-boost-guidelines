## Browser Automation

Use `agent-browser` for web automation. Run `agent-browser --help` for all commands.

Core workflow:

1. `agent-browser open <url>` - Navigate to page
2. `agent-browser snapshot -i` - Get interactive elements with refs (@e1, @e2)
3. `agent-browser click @e1` / `fill @e2 "text"` - Interact using refs
4. Re-snapshot after page changes

### Only use the chrome-dev-tools MCP server if the agent-browser cannot do what you need it to do. Agent-browser is faster and more reliable and chrome-dev-tools should only be used as a backup or needed for in-depth debugging.
