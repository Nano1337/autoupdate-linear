# Autoupdate Linear

Saves you from the manual slog of updating your linear issues. If you can do it, so can claude code!

Notes: 
- this must be used locally on Mac (as far as it's been tested). 
- This is an MVP, so please lmk or contribute if you want more features/better UX!

## Usage

Run setup below and then regular usage in claude code would be `/project:linear-standup path/standup-notes.md`.
- Standup notes markdown file should be based off the template `meeting_notes/LINEAR_UPDATE_TEMPLATE.md`.

## Setup

1. Install npm (search this up) and claude code using: 
```bash 
npm install -g @anthropic-ai/claude-code
claude
```
and authenticate using your MAX plan (do not use claude API). Cmd+C out of claude code once basic setup is done.

2. Run this command to add this alias to your `~/.zshrc`: 
```bash
grep -qxF 'alias yolo="claude --dangerously-skip-permissions"' ~/.zshrc \
  || echo 'alias yolo="claude --dangerously-skip-permissions"' >> ~/.zshrc
source ~/.zshrc
```
Don't worry, claude won't do anything unaligned (and if it does, the downsides are minimal), this is just so claude code doesn't ask permission to run every command

3. Add linear remote MCP using:
```bash 
claude mcp add --transport sse Linear https://mcp.linear.app/sse
```

4. Authenticate into Linear MCP by first running `yolo` to get your claude instance started and inside claude code, write `/mcp` to authenticate the Linear MCP from your side. 
You can test that the Linear MCP works by running inside claude code: `run this Linear:list_projects, Linear:list_teams`

## Custom Slash Commands

### Linear Standup Automation

Automate Linear ticket updates from natural language standup notes using advanced NLP parsing.

| Command | Description | Usage |
|---------|-------------|-------|
| `/project:linear-standup` | Parse standup notes and auto-create/update Linear tickets | `/project:linear-standup standup-notes.md` |
| `/project:linear-dry-run` | Preview Linear updates without making changes | `/project:linear-dry-run standup-notes.md` |
| `/project:linear-test` | Test Linear MCP connection and show workspace info | `/project:linear-test` |
| `/project:linear-workspace` | Show detailed Linear workspace structure | `/project:linear-workspace` |
| `/project:linear-fix` | Diagnose and fix Linear automation issues | `/project:linear-fix "error description"` |

**Features:**
- Intelligent parsing of natural language updates and TODOs
- Automatic @ mention mapping to Linear users
- Context-aware priority detection from urgency keywords
- Smart status updates based on progress indicators
- Comprehensive error handling and validation

**Requirements:** Linear MCP must be connected (`/mcp` to verify)
