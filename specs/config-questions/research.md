---
spec: config-questions
phase: research
created: 2026-01-22
---

# Research: config-questions

## Executive Summary

Adding config questions to `/ralph-specum:start` is straightforward - follow the existing pattern from branch strategy questions and goal interview. Three config options needed: `autoCommit`, `reviewEachTask`, `autoPushAndPR`. All three already exist in `.ralph-state.json` config object (see current spec state). Implementation requires: asking questions in start.md, storing in state, reading in spec-executor and implement commands.

## External Research

### Best Practices

N/A - This is internal plugin configuration, not a technology choice requiring external research.

### Prior Art

The codebase already has two established patterns for user configuration:

**1. Branch Strategy Questions** (start.md lines 42-97):
- Uses inline questions (not AskUserQuestion tool) with numbered options
- Asks different questions based on context (on default branch vs feature branch)
- Stores result implicitly via git commands

**2. Goal Interview Questions** (start.md lines 530-599):
- Uses AskUserQuestion tool with `questions` array and `options`
- Supports "Other" for free-text input
- Stores responses in `.progress.md` under "Goal Context" section
- Context passed to subagents via Task delegation prompt

**3. commitSpec Flag** (start.md lines 218-227):
- Uses CLI flags (`--commit-spec`, `--no-commit-spec`)
- Default logic based on mode (normal vs quick)
- Stored in `.ralph-state.json` as `commitSpec`
- Read by phase commands (research, requirements, design, tasks)

### Pitfalls to Avoid

- **AskUserQuestion NOT available in subagents** - questions must be in coordinator (start.md), not agents
- **Quick mode should skip questions** - existing pattern uses `--quick` flag check

## Codebase Analysis

### Existing Config Storage

Current `.ralph-state.json` already has a `config` object with the three fields:
```json
{
  "config": {
    "autoCommit": false,
    "reviewEachTask": true,
    "autoPushAndPR": false
  }
}
```

This structure was apparently added but never populated via user questions.

### Current State Schema

From `plugins/ralph-specum/schemas/spec.schema.json`, the state schema does NOT include the `config` object. Need to verify if schema needs updating or if config is intentionally not validated.

### Integration Points

| Config Option | Where Set | Where Read | Current Behavior |
|--------------|-----------|------------|------------------|
| `autoCommit` | start.md (to add) | spec-executor.md | Always commits per task |
| `reviewEachTask` | start.md (to add) | implement.md coordinator | Never pauses between tasks |
| `autoPushAndPR` | start.md (to add) | task-planner.md Phase 4 | Always creates PR in 4.2 |
| `commitSpec` | start.md (existing) | phase commands | Already working |

### Where Commits Happen

**spec-executor.md** lines 268-289:
```markdown
<mandatory>
ALWAYS commit spec files with every task commit. This is NON-NEGOTIABLE.
</mandatory>

- Each task = one commit
- Commit AFTER verify passes
- Use EXACT commit message from task
```

The spec-executor currently commits after EVERY task unconditionally.

### Where Task Review Pausing Should Happen

**implement.md** coordinator prompt section 8 "State Update":
- After TASK_COMPLETE, currently advances immediately
- Need to check `reviewEachTask` config and pause if true

### Where PR Creation Happens

**task-planner.md** lines 378-390 (Phase 4 template):
```markdown
- [ ] 4.2 Create PR and verify CI
  - **Do**:
    1. Push branch: `git push -u origin <branch-name>`
    4. Create PR using gh CLI: `gh pr create --title "<title>" --body "<summary>"`
```

Also in **templates/tasks.md** lines 142-156, same structure.

### AskUserQuestion Pattern

From `start.md` goal interview section:
```markdown
AskUserQuestion:
  questions:
    - question: "What problem are you solving with this feature?"
      options:
        - "Fixing a bug or issue"
        - "Adding new functionality"
        - "Improving existing behavior"
        - "Other"
```

Start.md has `allowed-tools: [Read, Write, Bash, Task, AskUserQuestion]` - AskUserQuestion is available.

## Related Specs

| Spec | Relevance | Reason | May Need Update |
|------|-----------|--------|-----------------|
| goal-interview | High | Already implemented AskUserQuestion in start.md | No - additive |
| implement-ralph-wiggum | Medium | Defines Ralph Loop execution model | Possibly - if reviewEachTask changes loop behavior |

## Quality Commands

| Type | Command | Source |
|------|---------|--------|
| Lint | Not found | No package.json in repo root |
| TypeCheck | Not found | No package.json |
| Test | Not found | No test infrastructure |
| Build | Not found | No build step (pure markdown/shell) |

**Local CI**: Plugin is pure markdown, no build/test commands. Version check CI exists for plugin files.

## Feasibility Assessment

| Aspect | Assessment | Notes |
|--------|------------|-------|
| Technical Viability | High | Simple extension of existing patterns |
| Effort Estimate | S | ~5-8 tasks, markdown-only changes |
| Risk Level | Low | Clear patterns to follow |

## Recommendations for Requirements

1. **Add config questions to start.md AFTER branch questions, BEFORE goal interview**
   - Questions should be optional (provide defaults)
   - Skip in `--quick` mode

2. **Use AskUserQuestion with 3 questions in one call**
   - Question 1: "Should Ralph commit automatically after each task?"
   - Question 2: "Should Ralph pause for review after each task?"
   - Question 3: "Should Ralph push and create PR when complete?"

3. **Store in existing config object in .ralph-state.json**
   - Schema: `config: { autoCommit, reviewEachTask, autoPushAndPR }`
   - All boolean values

4. **Modify spec-executor to check autoCommit**
   - If false, skip commit step but still stage files
   - Document what user needs to do (manual commit later)

5. **Modify implement.md coordinator to check reviewEachTask**
   - After TASK_COMPLETE verification passes
   - If reviewEachTask=true, set awaitingApproval=true and stop
   - User runs `/ralph-specum:implement` to continue

6. **Modify task-planner to conditionally include PR task**
   - If autoPushAndPR=false, replace 4.2 with "Manual: Push and create PR"
   - Or keep task but spec-executor checks config and skips PR creation

7. **Update .progress.md Configuration section**
   - Already shows config values (see current .progress.md)
   - Ensure start.md writes this section

## Open Questions

1. **Should config be editable mid-spec?** Current design sets once at start. User might want to change autoCommit partway through. Consider `/ralph-specum:config` command for future.

2. **What happens if autoCommit=false?** Spec-executor verification layer checks for uncommitted files. Need to adjust Layer 2 in coordinator or have user commit before next task.

## Sources

- `/home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/start.md` - Branch strategy and goal interview patterns
- `/home/omerr/repos/smart-ralph/plugins/ralph-specum/agents/spec-executor.md` - Commit discipline rules
- `/home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md` - Coordinator prompt, verification layers
- `/home/omerr/repos/smart-ralph/plugins/ralph-specum/agents/task-planner.md` - Phase 4 PR creation template
- `/home/omerr/repos/smart-ralph/specs/goal-interview/.progress.md` - AskUserQuestion learnings
