# Linear Update Automation Guide for Claude Code (Natural Language Format)

This document provides instructions for Claude Code to automatically parse natural language update templates and create/update Linear tickets using intelligent NLP-style parsing.

## Overview

When a user provides a free-form update template, Claude Code should:
1. Validate Linear MCP connection
2. Find existing tickets mentioned in "Updates Since Last Time"
3. Parse natural language for action items and TODOs
4. Extract @ mentions and map to team members
5. Identify priority indicators from context
6. Create or update Linear tickets with proper assignments
7. Provide a summary of all ticket actions

## Step 1: Initial Validation

### 1.1 Check Linear MCP Connection
```
User: "Please create Linear tickets from my update template"
Claude: [Uses /mcp command to verify Linear is connected]
```

If not connected, refer user to `docs/linear-mcp-setup.md`.

### 1.2 Read the Template
```
Claude: [Uses Read tool on LINEAR_UPDATE_TEMPLATE.md or user-specified file]
```

## Step 2: Parse Natural Language Template

### 2.1 Extract Project Context

Look for project indicators in natural language:
- Meeting headers (e.g., "Multimodal Team Weekly Sync")
- Project mentions in text
- Context clues from discussion topics

### 2.2 Parse "Updates Since Last Time" Section

Use intelligent parsing to find existing tickets:

```python
# Look for patterns like:
# - "Completed ML-123: Feature implementation"
# - "Made progress on the authentication ticket"
# - "Finished the API refactor (ticket #456)"
# - References to previous work items

patterns = [
    r'(?:ticket|issue|task)?\s*#?(\d+)',  # Direct ticket references
    r'([A-Z]+-\d+)',  # Standard ticket format (ML-123)
    r'(?:completed|finished|done with)\s+(.+?)(?:\s|$)',  # Completion indicators
    r'(?:progress on|working on|started)\s+(.+?)(?:\s|$)',  # Progress indicators
]
```

For each identified ticket:
1. Use `mcp__Linear__get_issue` to fetch current status
2. Prepare updates based on the description
3. Note completion status for summary

### 2.3 Extract @ Mentions

Parse the entire document for @ mentions:

```python
# Pattern: @FirstName or @FirstName LastName
mention_pattern = r'@(\w+(?:\s+\w+)?)'

# Extract all unique mentions
# Map to Linear users using fuzzy matching
```

### 2.4 Identify Natural Language Priority Indicators

Map contextual priority clues:

```python
priority_indicators = {
    1: ['urgent', 'ASAP', 'critical', 'blocker', 'immediately'],
    2: ['high priority', 'important', 'by EOD', 'today'],
    3: ['normal', 'standard', 'this week', 'by end of week'],
    4: ['low priority', 'nice to have', 'when possible', 'eventually']
}

# Also parse time-based priorities:
# "by tomorrow" → High
# "by Friday" → Calculate based on current day
# "next sprint" → Normal
```

## Step 3: Intelligent Action Item Extraction

### 3.1 Parse "Discussion Points" Section

Extract action items from natural conversation:

```python
# Look for action indicators:
action_patterns = [
    r'(?:we need to|should|will|must)\s+(.+)',
    r'(?:action item:|todo:|task:)\s*(.+)',
    r'(?:@\w+)\s+(?:will|should|needs to)\s+(.+)',
    r'(?:follow up on|investigate|implement|create|build)\s+(.+)'
]

# Extract context around each action item for better ticket descriptions
```

### 3.2 Parse "Immediate TODOs" Section

Handle various TODO formats:

```python
# Examples to parse:
# "- @Alice: Review and merge the authentication PR by EOD"
# "- Authentication system needs security review (@Bob, urgent)"
# "- Complete data migration script - @Charlie and @Dana"
# "- Fix the memory leak in image processing (high priority)"

todo_patterns = [
    r'-\s*@(\w+(?:\s+\w+)?)\s*:?\s*(.+)',  # - @Name: task
    r'-\s*(.+?)\s*\(@(\w+(?:\s+\w+)?)',     # - task (@Name
    r'-\s*(.+?)\s*-\s*@(\w+(?:\s+\w+)?)',   # - task - @Name
    r'-\s*(.+)',                             # - task (no assignee)
]
```

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

```python
# Strategy 1: Direct ticket ID search
if ticket_id_found:
    mcp__Linear__get_issue(id=ticket_id)
    
# Strategy 2: Search by description
if description_found:
    mcp__Linear__list_issues(
        query=description,
        teamId=team_id,
        updatedAt="-P7D"  # Last 7 days
    )
    
# Update existing tickets with:
# - Status changes (if completed)
# - Comments with progress updates
# - New information from the update
```

### 4.2 Create New Tickets from Action Items

Use parallel Task subagents for efficiency:

```python
# Group tasks by assignee for batch creation
tasks_by_assignee = group_tasks_by_assignee(parsed_tasks)

# Create parallel subagents
parallel_tasks = [
    {
        "description": f"Process {assignee}'s items",
        "prompt": f"""
        For {assignee} ({user_id}), handle these items:
        
        {format_tasks_with_context(tasks)}
        
        1. First check if any might be existing tickets using mcp__Linear__list_issues
        2. Create new tickets for items not found
        3. Include full context in descriptions
        4. Set priority based on: {priority_context}
        """
    }
    for assignee, tasks in tasks_by_assignee.items()
]
```

### 4.3 Enhanced Ticket Description Format

Include natural language context:

```markdown
## Summary
[Extracted task description with context]

## Context
From [Meeting/Update Date]:
[Relevant discussion points or background]

## Mentioned in Update
"[Quote the actual text that mentioned this task]"

## Requirements
[Parsed from natural language, may include]:
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
```

### 4.4 Handle Complex Scenarios

**Multiple Assignees on Same Task:**
```python
# Parse patterns like:
# "@Alice and @Bob: Collaborate on API design"
# "Security review (@Charlie, @Dana)"

if multiple_assignees:
    # Create main ticket for first assignee
    # Add others as collaborators or create linked sub-tasks
    # Include coordination notes in description
```

**Ambiguous Assignments:**
```python
# When no clear assignee but context suggests one:
# "The frontend team should update the dashboard"
# "Someone needs to fix the deployment script"

if no_direct_assignee:
    # Look for team mentions
    # Check previous similar tickets
    # Default to team lead or note as unassigned
```

## Step 5: Validation and Enhanced Summary

### 5.1 Comprehensive Validation

After all operations:
- Verify ticket creation/updates succeeded
- Check assignment accuracy (fuzzy matching results)
- Validate priority assignments made sense
- Ensure context was properly captured
- Flag any ambiguous items that need review

### 5.2 Provide Detailed Summary

Return a comprehensive summary:

```markdown
## Linear Update Processing Summary

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

**[Assignee 2 Name]**:
- [TICKET-ID]: [Title]
  Priority: [Priority]
  Context: From discussion about [topic]
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
```

## Error Handling

### Natural Language Parsing Challenges

1. **Ambiguous @ Mentions**
   ```
   Issue: "@Sarah" could be "Sarah Smith" or "Sarah Jones"
   Solution: 
   - Use fuzzy matching with full names
   - Check recent collaborators
   - Present options if multiple matches
   ```

2. **Vague Task Descriptions**
   ```
   Issue: "Fix the thing we discussed"
   Solution:
   - Include surrounding context in ticket
   - Reference meeting date and participants
   - Add note requesting clarification
   ```

3. **Conflicting Priority Signals**
   ```
   Issue: "Low priority but need it by tomorrow"
   Solution:
   - Deadline takes precedence
   - Note the conflict in ticket
   - Set as high priority with explanation
   ```

4. **Missing Ticket References**
   ```
   Issue: "Finished the authentication work"
   Solution:
   - Search recent tickets by keyword
   - Check assignee's recent tickets
   - Create follow-up note if can't find
   ```

5. **Complex Date Parsing**
   ```
   Issue: "Due by next Tuesday after the sprint ends"
   Solution:
   - Parse incrementally
   - Default to conservative estimates
   - Include original text in ticket
   ```

## Best Practices for Natural Language Processing

1. **Context is King**
   - Always capture 2-3 lines around any extracted item
   - Include meeting context and discussion topics
   - Quote original text in tickets for clarity

2. **Intelligent Defaults**
   - No priority mentioned → Check deadline first
   - No assignee → Look for contextual clues
   - Vague description → Include more context

3. **Fuzzy Matching Strategy**
   ```python
   # Priority order for matching:
   1. Exact @ mention match
   2. Partial name match with recent collaborators
   3. Team-based assignment from context
   4. Flag for manual review
   ```

4. **Preserve Original Intent**
   - Quote exact wording for requirements
   - Note interpretation decisions made
   - Include parsing confidence indicators

5. **Smart Ticket Grouping**
   - Related items → Link tickets
   - Multiple steps → Create sub-tasks
   - Shared work → Note collaboration

6. **Update vs Create Decision Tree**
   ```
   Found ticket reference? → Update
   Similar recent ticket exists? → Update or link
   Completely new work? → Create
   Unclear? → Create with reference note
   ```

7. **Handle Edge Cases Gracefully**
   - Multiple formats in same document
   - Mixed languages or informal text
   - Incomplete information

## Example Usage - Natural Language Template

```
User: "Create Linear tickets from my team update"

Claude: [Reads natural language update document]

## Sample Update Content:
"""
Multimodal Team Weekly Sync - Dec 24

Updates Since Last Time:
- Completed ML-123: Implemented the new image processing pipeline
- Made good progress on authentication system refactor, about 70% done
- Fixed the memory leak issue in the video encoder (ticket #456)

Discussion Points:
- @Alice brought up performance concerns with the current API design
- We need to investigate alternative caching strategies before next release
- The frontend team should coordinate with @Bob on the new UI components
- Action item: @Charlie will set up load testing for the new endpoints

Immediate TODOs:
- @Alice: Review and merge the authentication PR by EOD
- Complete data migration script - @Charlie and @Dana need to collaborate
- Fix deployment pipeline issues (urgent, blocking release)
- @Bob: Update documentation for API changes by Friday
"""

Claude Processing:
1. [Validates Linear connection]
2. [Finds project "Multimodal Team"]
3. [Identifies users: Alice, Bob, Charlie, Dana]
4. Updates existing tickets:
   - ML-123: Marked as complete
   - Ticket #456: Added completion comment
5. Creates new tickets:
   - "API Performance Investigation" → @Alice (High priority)
   - "Caching Strategy Research" → Unassigned (Medium)
   - "Load Testing Setup" → @Charlie (High, due this week)
   - "Authentication PR Review" → @Alice (Urgent, due today)
   - "Data Migration Script" → @Charlie & @Dana (High)
   - "Deployment Pipeline Fix" → Unassigned (Urgent)
   - "API Documentation Update" → @Bob (Medium, due Friday)
6. [Returns comprehensive summary with all actions taken]
```

## Advanced Natural Language Patterns

### Detecting Implicit Actions

```python
# Patterns that suggest action items without explicit TODOs:
implicit_patterns = [
    r'(?:we should|need to|have to|must)\s+(.+)',
    r'(?:someone needs to|who can|anyone able to)\s+(.+)',
    r'(?:investigate|look into|research|explore)\s+(.+)',
    r'(?:follow up on|check on|verify)\s+(.+)',
    r'(?:deadline:|due:|by:|before:)\s*(.+)',
]

# Context clues for urgency without explicit priority:
urgency_context = [
    'blocking', 'bottleneck', 'critical path',
    'customer waiting', 'production issue',
    'regression', 'security', 'compliance'
]
```

### Handling Meeting Notes Style

```python
# Common meeting note formats:
# "AI: @person - description" (Action Item)
# "[Person] to do [task]"
# "Owner: [Person], Task: [Description]"
# "TODO(@person): description"
# "Decision: [Person] will [action]"

meeting_patterns = [
    r'AI:\s*@?(\w+)\s*[-:]?\s*(.+)',
    r'\[(\w+)\]\s+to\s+(.+)',
    r'Owner:\s*(\w+).*?Task:\s*(.+)',
    r'TODO\(@?(\w+)\):\s*(.+)',
    r'(\w+)\s+will\s+(.+)',
]
```

### Smart Ticket Title Generation

When task descriptions are vague, generate clear titles:
- "Fix the thing" → "Fix [Component] Issue Discussed in [Meeting Date]"
- "Update docs" → "Update [Product] Documentation for [Feature]"
- "Performance work" → "Optimize [System] Performance per [Date] Discussion"

## Template Versions

This guide is compatible with:
- Natural language update templates (free-form)
- LINEAR_UPDATE_TEMPLATE.md v2.0 (natural language format)
- Linear MCP API v1.0
- Claude Code with MCP support

---

**Last Updated**: 2024-12-24
**Guide Version**: 2.0 (Natural Language Support)