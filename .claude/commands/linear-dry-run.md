---
allowed-tools: Linear:*
description: Preview Linear updates without making changes (dry run mode)
---

# Linear Standup Preview (Dry Run)

Parse standup notes and show what would be updated in Linear, without making actual changes.

## Input
Standup notes file: @$ARGUMENTS

## Analysis Mode
Perform all parsing and analysis but DO NOT create or update any tickets.

## Output
Show:
- What existing issues would be updated
- What comments would be added  
- What new issues would be created
- What status changes would be made
- What assignees would be updated
- Any potential problems or conflicts

## Validation
- Check that all @ mentions map to valid Linear users
- Verify project names exist
- Confirm status mappings are valid
- Flag any ambiguous or unclear items

This is useful for testing your standup template format before committing to actual Linear updates.
