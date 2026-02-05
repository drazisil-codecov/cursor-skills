---
name: track-pr-test-plan
description: Create Linear tracking issues for PR test plans and post-deployment validation. Use when a PR contains a test plan, checklist, or validation steps that need to be tracked after merge, or when the user asks how to ensure test plans aren't lost.
---

# Track PR Test Plans

Ensures PR test plans and validation checklists don't get lost after merge by creating Linear tracking issues.

## When to Use

Apply this skill when:
- A PR has a "Test Plan" section with unchecked items
- Post-deployment validation is required
- Manual testing steps need to be tracked
- The user asks how to ensure test plans are followed up on

## Process

### Step 1: Extract Test Plan from PR

Review the PR description for test plan sections:

```bash
# Get PR details
gh pr view <pr-number> --json body,title,number,url
```

Look for sections like:
- "Test Plan"
- "Validation Steps"
- "Post-Deployment Checklist"
- "Verification"

### Step 2: Determine Tracking Strategy

**For pre-merge validation**: Testing should be completed before merge. No tracking issue needed.

**For post-merge validation**: Create a Linear issue if the test plan includes:
- Staging/production deployment verification
- Performance monitoring
- Feature flag rollout steps
- User feedback collection
- Multi-environment testing

### Step 3: Get PR Author, Team, and Current Sprint

**Get PR author**:

```bash
# Get PR author's GitHub login
gh pr view <pr-number> --json author --jq '.author.login'
```

Then find their Linear user account:

```javascript
// Search for Linear user by GitHub username or name
await user-linear-list_users({
  query: "<github-username>"
})
```

**Determine the team**:

Check if the PR references a Linear issue:

```bash
# Look for Linear issue references in PR body
gh pr view <pr-number> --json body | grep -i "CODECOV-\|LINEAR-\|refs"
```

If a Linear issue is referenced, get its team:

```javascript
// Get the original issue details
await user-linear-get_issue({
  query: "<issue-identifier>" // e.g., "CODECOV-123"
})
// Extract team from issue.team or issue.teamId
```

If no Linear issue exists, determine team by:
- Code area/directory being changed
- Component ownership
- Ask the user if unclear

**Get current cycle for that team**:

```javascript
// List current cycles for the team
await user-linear-list_cycles({
  teamId: "<team-id-from-original-issue>",
  type: "current"
})
```

### Step 4: Create Linear Tracking Issue

Create a Linear issue with:

**Title format**: `Validate [feature/fix] after PR #<number> deployment`

**Description structure**:
```markdown
## Background

[Brief context about what the PR does]

**Related:**
- PR: [PR URL]
- Original issue: [Linear issue if exists]

## Validation Steps

### Staging
- [ ] [Specific validation step]
- [ ] [Specific validation step]

### Production
- [ ] [Specific validation step]
- [ ] [Specific validation step]
- [ ] [Monitor for X period]

## Success Criteria

✅ [Specific measurable outcome]
✅ [Specific measurable outcome]

## Rollback Plan

If validation fails:
1. [Rollback step]
2. [Investigation step]
```

**Set appropriate metadata**:
- **Priority**: Match the original issue's priority (High for production fixes)
- **Assignee**: PR author (from Step 3)
- **Team**: Same team as the original Linear issue (from Step 3)
- **Cycle**: Current active cycle for that team (from Step 3)

```javascript
// Example Linear issue creation
await user-linear-create_issue({
  title: "Validate Sentry worker tracing after PR #680 deployment",
  description: "[See template above]",
  priority: 2, // Match original issue priority
  assignee: "<pr-author-linear-username>", // From Step 3
  team: "<team-from-original-issue>", // From Step 3
  cycle: "<current-cycle-for-that-team>" // From Step 3
})
```

### Step 5: Link Issue to PR

Add a comment to the PR linking to the tracking issue:

```markdown
## 📋 Post-Deployment Validation

Created tracking issue to ensure the test plan is followed after deployment:

**[CODECOV-57: Validate feature after deployment](https://linear.app/...)**

This tracks:
- ✅ Staging validation
- ✅ Production monitoring
- ✅ Success criteria and rollback plan
```

## Best Practices

### Issue Timing

**Create before merge** for visibility to:
- PR reviewers (see validation is planned)
- Deployment team (know what to watch for)
- Original author (responsibility is clear)

### Make Issues Actionable

- Use checkboxes for trackable steps
- Include specific metrics to measure (not "verify it works")
- Add links to monitoring dashboards or tools
- Specify time windows (e.g., "monitor for 1 hour")

### Set Clear Ownership

- **Default to PR author**: They understand the change best and should validate it
- **Use same team as original issue**: Keeps the work in the right team's backlog
- **Add to current sprint**: Ensures validation is planned and doesn't slip
- @ mention stakeholders who should be notified
- Set due date tied to expected deployment date

If the PR author isn't available (vacation, left team):
1. Assign to tech lead or person handling deployment
2. Add note explaining why not assigned to author

If no original Linear issue exists:
1. Determine team by code ownership or component area
2. Ask the user if unclear

### Add Context for Future Reference

- Link back to PR for full context
- Reference original issue/ticket
- Document why validation is important
- Include rollback criteria

## Complete Workflow Example

Here's a full example showing PR author lookup, team extraction, and sprint assignment:

```bash
# 1. Get PR details and check for Linear issue reference
gh pr view 680 --json author,url,title,body

# 2. Extract referenced Linear issue (e.g., "Relates to: CCMRG-2037")
# Get original issue details
user-linear-get_issue({ query: "CCMRG-2037" })
# Returns: { 
#   id: "issue-123", 
#   team: "Codecov Engineering", 
#   teamId: "codecov-team-id",
#   priority: { value: 2 }
# }

# 3. Find PR author in Linear
user-linear-list_users({ query: "drazisil-codecov" })
# Returns: { id: "abc123", name: "Joe Becher", email: "joe@..." }

# 4. Get current cycle for the same team as original issue
user-linear-list_cycles({ teamId: "codecov-team-id", type: "current" })
# Returns: { id: "cycle-xyz", name: "Sprint 24", ... }

# 5. Create tracking issue with same team
user-linear-create_issue({
  title: "Validate Sentry worker tracing after PR #680 deployment",
  description: "## Background\n\nPR #680 removes Sentry re-initialization...",
  priority: 2, // Match original issue priority
  assignee: "abc123", // PR author's Linear ID
  team: "Codecov Engineering", // Same team as original issue
  cycle: "cycle-xyz" // Current sprint for that team
})

# 6. Link back to PR
gh pr comment 680 --body "📋 Created tracking issue: [CODECOV-57](https://linear.app/...)"
```

## Examples

### Example 1: Performance Change

PR makes database query optimization.

**Linear issue**:
```markdown
## Background
PR #123 optimizes the users table query by adding an index.

## Validation Steps

### Staging
- [ ] Deploy to staging
- [ ] Run load test: `ab -n 1000 -c 10 https://staging.../users`
- [ ] Compare p95 latency vs baseline (baseline: 450ms, target: <200ms)

### Production
- [ ] Deploy to production
- [ ] Monitor Datadog dashboard for 2 hours
- [ ] Compare error rates vs 24h average
- [ ] Check slow query log for the optimized query

## Success Criteria
✅ p95 latency < 200ms
✅ No increase in error rate
✅ Query no longer in slow query log
```

### Example 2: Feature Flag Rollout

PR adds new feature behind a flag.

**Linear issue**:
```markdown
## Background
PR #456 adds new billing flow behind `new_billing_v2` flag.

## Validation Steps

### Staging
- [ ] Enable flag for test accounts
- [ ] Complete full purchase flow 5 times
- [ ] Verify receipts in admin panel

### Production
- [ ] Enable for 5% of users
- [ ] Monitor for 1 hour: error rates, conversion rate
- [ ] Enable for 50% of users
- [ ] Monitor for 24 hours
- [ ] Full rollout

## Success Criteria
✅ Conversion rate >= baseline
✅ Payment processing errors < 0.1%
✅ No increase in support tickets
```

### Example 3: Bug Fix Verification

PR fixes a specific error condition.

**Linear issue**:
```markdown
## Background
PR #789 fixes NPE in user profile endpoint when avatar is null.

## Validation Steps

### Staging
- [ ] Deploy to staging
- [ ] Create test user with null avatar
- [ ] Access profile page 10 times
- [ ] Verify no errors in Sentry

### Production
- [ ] Deploy to production
- [ ] Monitor Sentry for "NullPointerException" in ProfileController
- [ ] Confirm error volume drops to 0 within 1 hour
- [ ] Check that existing null avatars render correctly

## Success Criteria
✅ Zero NPE errors in ProfileController after deployment
✅ All profiles with null avatars render correctly
```

## Anti-Patterns

### ❌ Don't Create Issues for Unit Tests

If the PR test plan is just "run unit tests", don't create a tracking issue. CI should enforce this.

### ❌ Don't Use Vague Success Criteria

**Bad**: "Make sure it works"
**Good**: "API response time < 200ms, error rate < 0.1%"

### ❌ Don't Skip Rollback Plans

Always include what to do if validation fails. The 3am-deployment-gone-wrong person will thank you.

### ❌ Don't Lose Context

Always link back to the PR and original issue. Future you shouldn't have to detective work.

## Integration with Existing Workflows

### With PR Review Process

1. During PR review, check if test plan requires post-merge tracking
2. If yes, create Linear issue before approving
3. Mention tracking issue in approval comment

### With Deployment Process

1. Check Linear for validation issues related to deployment
2. Complete validation steps after deployment
3. Update issue with results
4. Close issue only after all steps pass

### With Monitoring

Link relevant dashboards in the Linear issue:
- Datadog/Grafana links
- Sentry projects/issues
- Error tracking tools
- Business metrics dashboards

## Troubleshooting

**Q: Should I create the issue before or after merge?**
A: Before merge. This makes validation requirements visible to reviewers and ensures it's not forgotten.

**Q: What if the PR is already merged?**
A: Still create the tracking issue. Better late than never.

**Q: Who should I assign the issue to?**
A: Default to PR author. Get their GitHub username from the PR, then search Linear users to find their account. If they're not in Linear or unavailable, assign to the tech lead or deployment owner.

**Q: What if I can't find the current sprint/cycle?**
A: If no active cycle exists, skip the cycle assignment. The issue will still be created and show up in the team's backlog.

**Q: What priority should I use?**
A: Match the PR's urgency. Production hotfixes = High/Urgent. Feature work = Normal/Low.

**Q: Should I assign to current sprint even if it's already full?**
A: Yes. Validation is part of delivering the feature. If sprint capacity is an issue, that's a planning conversation for the team.

**Q: What if the PR doesn't reference a Linear issue?**
A: Determine the team by:
1. Looking at which code area/directory is being changed
2. Checking component ownership (e.g., worker changes → Codecov team)
3. Asking the user which team should track the validation
As a last resort, assign to the same team as the PR author.
