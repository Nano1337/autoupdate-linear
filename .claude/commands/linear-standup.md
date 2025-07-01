---
allowed-tools: Linear:*, Bash(*)
description: Parse standup notes and update Linear issues automatically using advanced natural language processing
---

# Linear Standup Update Processor

## Context
- Input standup notes: @$ARGUMENTS
- Current Linear workspace: !`echo "Fetching Linear workspace data..."`

## Advanced Natural Language Parsing Instructions

You are an expert at parsing natural language standup notes and updating Linear issues. You may use as many parallel subagent `Tasks()` to make your updates. Follow the comprehensive instructions below:

# Linear Update Automation Guide for Claude Code (Natural Language Format)

This document provides instructions for Claude Code to automatically parse natural language update templates and create/update Linear tickets using intelligent NLP-style parsing.

## Overview

When processing a standup update template, you should:
1. Validate Linear MCP connection
2. Find existing tickets mentioned in "Updates Since Last Time"
3. Parse natural language for action items and TODOs
4. Extract @ mentions and map to team members
5. Identify priority indicators from context
6. Create or update Linear tickets with proper assignments
7. Provide a summary of all ticket actions

## Step 1: Initial Validation

### 1.1 Check Linear MCP Connection
First verify Linear connection by listing teams and projects to ensure MCP is working.

### 1.2 Read the Template
Process the standup notes provided as input via `$ARGUMENTS`.

## Step 2: Parse Natural Language Template

### 2.1 Extract Project Context

Look for project indicators in natural language:
- Meeting headers (e.g., "Multimodal Team Weekly Sync")
- Project mentions in text
- Context clues from discussion topics

### 2.2 Parse "Updates Since Last Time" Section

Use intelligent parsing to find existing tickets:

Look for patterns like:
- "Completed ML-123: Feature implementation"
- "Made progress on the authentication ticket"
- "Finished the API refactor (ticket #456)"
- References to previous work items

Patterns to detect:
- Direct ticket references: `#123`, `ML-123`, `ticket 456`
- Completion indicators: "completed", "finished", "done with", "wrapped up"
- Progress indicators: "progress on", "working on", "started", "heads down"

For each identified ticket:
1. Use `Linear:get_issue` to fetch current status
2. Prepare updates based on the description
3. Note completion status for summary

### 2.3 Extract @ Mentions

Parse the entire document for @ mentions:
- Pattern: `@FirstName` or `@FirstName LastName`
- Extract all unique mentions
- Map to Linear users using fuzzy matching with `Linear:list_users`

### 2.4 Identify Natural Language Priority Indicators

Map contextual priority clues:

**Priority 1 (Urgent)**: 'urgent', 'ASAP', 'critical', 'blocker', 'immediately'
**Priority 2 (High)**: 'high priority', 'important', 'by EOD', 'today'
**Priority 3 (Normal)**: 'normal', 'standard', 'this week', 'by end of week'
**Priority 4 (Low)**: 'low priority', 'nice to have', 'when possible', 'eventually'

Also parse time-based priorities:
- "by tomorrow" → High
- "by Friday" → Calculate based on current day
- "next sprint" → Normal

## Step 3: Intelligent Action Item Extraction

### 3.1 Parse "Discussion Points" Section

Extract action items from natural conversation:

Look for action indicators:
- "we need to", "should", "will", "must" + action
- "action item:", "todo:", "task:" + description
- "@person will/should/needs to" + action
- "follow up on", "investigate", "implement", "create", "build" + object

Extract context around each action item for better ticket descriptions.

### 3.2 Parse "Immediate TODOs" Section

Handle various TODO formats:

Examples to parse:
- "- @Alice: Review and merge the authentication PR by EOD"
- "- Authentication system needs security review (@Bob, urgent)"
- "- Complete data migration script - @Charlie and @Dana"
- "- Fix the memory leak in image processing (high priority)"

TODO patterns:
- `- @Name: task` or `- @Name task`
- `- task (@Name` or `- task - @Name`
- `- task` (no assignee)

### 3.3 Smart Context Extraction

For each identified task:
1. Extract surrounding context (2-3 lines before/after)
2. Identify related discussion points
3. Find any mentioned documents or links
4. Determine if it's related to an existing ticket

### 3.4 Infer Missing Information

When information is implicit:
- **No assignee**: Check if task context mentions someone
- **No priority**: Analyze urgency words and deadlines
- **No due date**: Parse relative dates ("next week", "by Friday")
- **Vague descriptions**: Include surrounding context

## Step 4: Linear Operations

### 4.1 Find and Update Existing Tickets

For tickets mentioned in "Updates Since Last Time":

**Strategy 1**: Direct ticket ID search
Linear:get_issue id="ticket-id"

**Strategy 2**: Search by description
Linear:list_issues query="description keywords" teamId="team-id"

Update existing tickets with:
- Status changes (if completed)
- Comments with progress updates
- New information from the update

### 4.2 Create New Tickets from Action Items

For each new action item:

1. **Check if similar ticket exists** using `Linear:list_issues` with relevant query
2. **Create new ticket** if not found using `Linear:create_issue`
3. **Include full context** in descriptions
4. **Set priority** based on detected indicators

### 4.3 Enhanced Ticket Description Format

Include natural language context:

```markdown
## Summary
[Extracted task description with context]

## Context
From Standup [Date]:
[Relevant discussion points or background]

## Mentioned in Update
"[Quote the actual text that mentioned this task]"

## Requirements
[Parsed from natural language]:
- Main objective
- Any specific requirements mentioned
- Technical details discussed

## Priority Rationale
[Why this priority was assigned - urgency words, deadlines, etc.]

## Related Work
- Previous tickets: [If found]
- Related discussions: [From template]

## Source
Created from: [Template section where this was found]
Original mention by: [Who brought it up if clear]
4.4 Handle Complex Scenarios
Multiple Assignees on Same Task:

Create main ticket for first assignee
Add others as collaborators in comments
Include coordination notes in description

Ambiguous Assignments:

Look for team mentions or context clues
Check previous similar tickets
Default to unassigned with note for clarification

Step 5: Validation and Enhanced Summary
5.1 Comprehensive Validation
After all operations:

Verify ticket creation/updates succeeded
Check assignment accuracy (fuzzy matching results)
Validate priority assignments made sense
Ensure context was properly captured
Flag any ambiguous items that need review

5.2 Provide Detailed Summary
Return a comprehensive summary:
markdown## Linear Update Processing Summary

**Update Document**: [Filename]
**Project**: [Project Name] ([URL])
**Processing Date**: [Date]

### Existing Tickets Updated

**Completed Tickets**:
- [TICKET-ID]: [Title] - Marked as complete
  Previous status: [Status] → Done
  URL: [Linear URL]

**Progress Updates**:
- [TICKET-ID]: [Title] - Added progress comment
  Update: "[Summary of progress]"
  URL: [Linear URL]

### New Tickets Created

**[Assignee 1 Name]**:
- [TICKET-ID]: [Title]
  Priority: [Priority] (detected: "urgent" in context)
  Due: [Date] (parsed from "by Friday")
  Source: Immediate TODOs section
  URL: [Linear URL]

**Unassigned/Review Needed**:
- [TICKET-ID]: [Title] - No clear assignee found
  Context: "[Original text]"
  URL: [Linear URL]

### Parsing Notes
- Found [N] @ mentions, matched [M] to Linear users
- Detected [N] priority indicators
- Identified [N] existing ticket references
- [N] items needed context inference

### Summary Statistics
- Total tickets updated: [N]
- Total tickets created: [N]
- High priority items: [N]
- Items due this week: [N]
- Items needing review: [N]
Error Handling
Natural Language Parsing Challenges

Ambiguous @ Mentions

Use fuzzy matching with full names from Linear:list_users
Check recent collaborators
Present options if multiple matches


Vague Task Descriptions

Include surrounding context in ticket
Reference meeting date and participants
Add note requesting clarification


Conflicting Priority Signals

Deadline takes precedence
Note the conflict in ticket
Set as high priority with explanation


Missing Ticket References

Search recent tickets by keyword using Linear:list_issues
Check assignee's recent tickets
Create follow-up note if can't find


Complex Date Parsing

Parse incrementally
Default to conservative estimates
Include original text in ticket



Best Practices for Natural Language Processing

Context is King

Always capture 2-3 lines around any extracted item
Include meeting context and discussion topics
Quote original text in tickets for clarity


Intelligent Defaults

No priority mentioned → Check deadline first
No assignee → Look for contextual clues
Vague description → Include more context


Preserve Original Intent

Quote exact wording for requirements
Note interpretation decisions made
Include parsing confidence indicators


Smart Ticket Grouping

Related items → Link tickets in comments
Multiple steps → Create with detailed descriptions
Shared work → Note collaboration


Update vs Create Decision Tree

Found ticket reference? → Update
Similar recent ticket exists? → Update or link
Completely new work? → Create
Unclear? → Create with reference note



Advanced Natural Language Patterns
Detecting Implicit Actions
Patterns that suggest action items without explicit TODOs:

"we should", "need to", "have to", "must" + action
"someone needs to", "who can", "anyone able to" + action
"investigate", "look into", "research", "explore" + object
"follow up on", "check on", "verify" + object

Context clues for urgency without explicit priority:

'blocking', 'bottleneck', 'critical path'
'customer waiting', 'production issue'
'regression', 'security', 'compliance'

Smart Ticket Title Generation
When task descriptions are vague, generate clear titles:

"Fix the thing" → "Fix [Component] Issue Discussed in [Date] Standup"
"Update docs" → "Update [Product] Documentation for [Feature]"
"Performance work" → "Optimize [System] Performance per [Date] Discussion"

Linear MCP Commands Reference
Use these specific commands:
bash# Get workspace info
Linear:list_teams
Linear:list_projects

# Find project
Linear:get_project "Project Name"

# User management
Linear:list_users
Linear:get_user "User Name"

# Issue operations
Linear:list_issues query="keywords" projectId="uuid"
Linear:get_issue id="issue-id"
Linear:create_issue title="Title" description="Description" teamId="team-uuid" assigneeId="user-uuid" priority=2
Linear:update_issue id="issue-uuid" stateId="state-uuid"
Linear:create_comment issueId="issue-uuid" body="Comment text"

# Get status options
Linear:list_issue_statuses teamId="team-uuid"
Your Task

Validate Linear connection by checking teams/projects
Parse the standup notes using the natural language patterns above
Find existing tickets mentioned in updates
Extract action items from all sections
Map @ mentions to Linear users
Create/update tickets as appropriate
Provide comprehensive summary of all actions

Start by getting the Linear workspace structure, then systematically process each section of the standup notes using the intelligent parsing methods described above.

## Usage Instructions

### Setup
1. Save the above content to `.claude/commands/linear-standup.md`
2. Make sure Linear MCP is connected (`/mcp` to check status)
3. Test with a sample standup notes file

### Running the Command
```bash
# Process standup notes
/project:linear-standup standup-2025-07-01.md

# Or with full path
/project:linear-standup @./meeting-notes/standup-2025-07-01.md

Expected Output
The command will:

Parse your standup template
Show what Linear projects/issues it found
List the updates it's making
Provide a summary of changes
