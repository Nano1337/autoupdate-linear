# Autoupdate Linear

Saves you from the manual slog of updating your linear issues. If you can do it, so can claude code!

Notes: 
- this must be used locally on Mac (as far as it's been tested). 
- This is an MVP, so please lmk or contribute if you want more features/better UX!

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

5. 
