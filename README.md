# 🚀 Autoupdate Linear

> Transform your meeting notes into Linear tickets automatically with Claude Code's intelligent parsing

## Overview

Autoupdate Linear eliminates the manual work of creating and updating Linear issues from standup meetings. Simply paste your meeting notes in a natural language format, and Claude Code will:

- 🎯 Extract action items and TODOs
- 👤 Map team member mentions to Linear users
- 📊 Set priorities based on urgency keywords
- 🔄 Update existing tickets with progress
- ✨ Create new tickets with full context

**Platform Support:** macOS (tested)  
**Status:** MVP - Contributions welcome!

## 📋 Prerequisites

- 🍎 macOS
- 🤖 Claude MAX subscription (not API)
- 📝 Granola AI app (or similar) for meeting transcripts
- 📊 Linear account with MCP access

## 🛠️ Setup Guide

### 1. Install Claude Code
First, ensure you have Node.js and npm installed, then:

```bash 
npm install -g @anthropic-ai/claude-code
claude
```

📌 **Important:** Authenticate with your Claude MAX subscription (not API key). Exit with `Cmd+C` after initial setup.

### 2. Configure Auto-permissions
Add this convenience alias to skip permission prompts:

```bash
grep -qxF 'alias yolo="claude --dangerously-skip-permissions"' ~/.zshrc \
  || echo 'alias yolo="claude --dangerously-skip-permissions"' >> ~/.zshrc
source ~/.zshrc
```

💡 **Note:** This alias allows Claude Code to run commands without asking permission each time. It's safe for local development use.

### 3. Connect Linear MCP
Add the Linear Model Context Protocol connection:

```bash 
claude mcp add --transport sse Linear https://mcp.linear.app/sse
```

### 4. Authenticate Linear
1. Start Claude Code with: `yolo`
2. Inside Claude Code, type `/mcp` to authenticate Linear
3. Test the connection with:
   ```
   run this Linear:list_projects, Linear:list_teams
   ```

## 🚀 Quick Start

### 1. Record Your Meeting
Use Granola AI (or similar tool) to capture your meeting transcript

### 2. Create Meeting Notes
Copy the template from `meeting_notes/LINEAR_UPDATE_TEMPLATE.md` and paste your transcript

### 3. Run the Command
In Claude Code, execute:
```
/project:linear-standup meeting_notes/your-standup-2025-07-01.md
```

### 4. Review Results
Claude will show you:
- ✅ Tickets created
- 🔄 Tickets updated
- 👥 User assignments
- 📊 Priority levels set

## 🎯 Available Commands

| Command | Description | Example |
|---------|-------------|---------|
| **📝 linear-standup** | Process meeting notes and update Linear | `/project:linear-standup notes.md` |
| **👀 linear-dry-run** | Preview changes without updating Linear | `/project:linear-dry-run notes.md` |
| **🔍 linear-test** | Verify Linear connection | `/project:linear-test` |
| **🏢 linear-workspace** | Display workspace structure | `/project:linear-workspace` |
| **🔧 linear-fix** | Troubleshoot issues | `/project:linear-fix "error msg"` |

## ✨ Key Features

### Intelligent Parsing
- 🧠 **Natural Language Processing**: Understands conversational meeting notes
- 🎯 **Action Item Detection**: Finds TODOs even when not explicitly marked
- 👤 **Smart User Mapping**: Matches @mentions to Linear users automatically
- ⏰ **Priority Detection**: Recognizes urgency keywords (ASAP, EOD, urgent)
- 📅 **Date Parsing**: Converts "next Friday" to actual dates

### Ticket Management
- 🔄 Updates existing tickets mentioned in notes
- ✨ Creates new tickets from action items
- 💬 Adds context and discussion points
- 🏷️ Sets appropriate labels and priorities
- 📎 Links related tickets together

## 📚 Meeting Notes Template

The template (`meeting_notes/LINEAR_UPDATE_TEMPLATE.md`) includes sections for:

1. **Project Information** - Meeting metadata
2. **Updates Since Last Time** - Progress on existing work
3. **Discussion Points** - Meeting transcript from Granola AI
4. **Immediate TODOs** - Action items with assignees
5. **Notes for Next Time** - Follow-up items

💡 **Pro Tip:** Write naturally! The parser understands conversational language, so focus on capturing the discussion rather than formatting.

## 🔧 Troubleshooting

### Common Issues

**Linear MCP not connected**
- Run `/mcp` in Claude Code to reconnect
- Verify with `/project:linear-test`

**User not found**
- Check spelling of @mentions
- Use `/project:linear-workspace` to see all users
- The parser uses fuzzy matching but exact names work best

**Tickets not creating**
- Ensure you have a valid project selected
- Check team permissions in Linear
- Use `/project:linear-dry-run` to preview first

## 🤝 Contributing

This is an MVP - contributions are welcome! Feel free to:
- Report bugs
- Suggest features
- Submit pull requests
- Improve documentation

## 📄 License

MIT
