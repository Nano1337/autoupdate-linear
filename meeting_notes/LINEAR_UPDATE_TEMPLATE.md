# Team Meeting Notes

## Project Information

**Project**: [Your project name in Linear]  
**Date**: [Meeting date]  
**Attendees**: [List everyone who attended]

---

## Updates Since Last Time

*Write naturally about what you've been working on. No need for bullet points if you don't want them - just share your progress like you're talking to the team.*

### Example entries:

**Sarah**: Finally wrapped up the authentication refactor! It took longer than expected because we discovered some edge cases with SSO, but it's all working now. Also started looking into the performance issues on the dashboard - turns out we're making way too many API calls.

**Mike**: Been heads down on the data pipeline. Got the ETL process running smoothly, processing about 2M records per hour now. Hit a snag with the deduplication logic but Alex helped me sort it out. Should be ready to deploy to staging by Thursday.

**Alex**: Mostly been in support mode this week - helped Mike with the dedup issue and reviewed a bunch of PRs. Also investigated that customer bug report about exports timing out. Turns out it's related to the dashboard performance issues Sarah mentioned.

---

## Discussion Points

*Paste your meeting notes from Granola, Otter, or whatever you use. Keep the natural flow of conversation.*

### Example:

**Performance Issues Discussion**
- Sarah: "The dashboard is making 50+ API calls on initial load"
- Mike: "Can we batch those? Or maybe implement GraphQL?"
- Alex: "Let's start with batching - simpler to implement"
- Decision: Sarah will create a batching endpoint, target 5 API calls max

**Customer Feedback Review**
- Three customers reported export timeouts
- All were trying to export >100k records
- Quick fix: Add progress indicator and increase timeout
- Long-term: Implement async exports with email notification

**Sprint Planning**
- Agreed to focus on performance improvements this sprint
- Mike suggested we add monitoring before making changes
- Sarah will set up DataDog dashboards

---

## Immediate TODOs

*Tag people naturally for tasks that came up during the meeting. Write it like you're sending a Slack message.*

@Sarah needs to create the batching endpoint for dashboard API calls - let's aim for reducing those 50 calls down to 5 max

@Mike and @Alex should pair on setting up DataDog monitoring before we start the performance work - we need baseline metrics

@Alex to implement the quick fix for export timeouts (progress indicator + timeout increase) - customers are waiting on this

@Sarah will write up a quick doc on the performance findings and share with the team

@Mike has to finish the data pipeline deployment to staging by Thursday - @Alex can you review once it's up?

@Everyone: Please update your Linear tickets with current status before EOD

---

## Notes for Next Time

*Anything we should remember for the next meeting*

- Review DataDog metrics to see if batching helped
- Check in on customer export issues
- Mike mentioned wanting to discuss moving to Kubernetes - add to next week's agenda
- Sarah's PTO next Friday - plan accordingly

---

*Template tip: Write naturally! This gets parsed into Linear tickets automatically, so just focus on capturing the conversation and action items clearly.*