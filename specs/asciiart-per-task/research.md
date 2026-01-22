---
spec: asciiart-per-task
phase: research
created: 2026-01-22
---

# Research: asciiart-per-task

## Executive Summary

Adding ASCII art diagrams when `reviewEachTask=true` requires modifying the pause message output in `implement.md` coordinator (section 8, State Update). The coordinator has full context: current/next task descriptions, progress history, and design.md architecture. Diagrams can be generated inline by Claude without external tools.

## External Research

### ASCII Art Diagram Best Practices

**Box Drawing Characters** ([Wikipedia](https://en.wikipedia.org/wiki/Box-drawing_characters)):
- Unicode U+2500 range provides clean single-line characters
- Corners: `+` (ASCII) or `+-` combinations
- Standard set: `+-|` (maximum compatibility)
- Enhanced set: `+` for corners, `-` for horizontal, `|` for vertical

**Popular Patterns** ([GitHub Gist Examples](https://gist.github.com/dsample/79a97f38bf956f37a0f99ace9df367b9)):
```
Simple Box:          Connected:          Flow:
+-------+           +---+   +---+       +---+
|       |           |   |-->|   |       | A |
+-------+           +---+   +---+       +---+
                                          |
                                          v
                                        +---+
                                        | B |
                                        +---+
```

**Tools** ([Baeldung](https://www.baeldung.com/linux/shell-ascii-diagrams)):
- External tools (Graph-Easy, Diagon, PlantUML) not needed
- Claude can generate diagrams inline from context
- No dependency required

### Diagram Types for Task Progress

| Type | Use Case | Example |
|------|----------|---------|
| Linear flow | Sequential tasks | `[Done] --> [Current] --> [Next]` |
| Component diagram | Architecture changes | Boxes with connections |
| Progress bar | Overall completion | `[=====>    ] 60%` |
| State machine | State changes | States with transitions |

## Codebase Analysis

### Where Pause Happens

**File**: `plugins/ralph-specum/commands/implement.md`
**Section**: 8. State Update (lines 423-460)
**Condition**: `reviewEachTask=true AND taskIndex < totalTasks`

Current pause output:
```
TASK COMPLETE - PAUSED FOR REVIEW

Task $previousTaskIndex completed successfully.
Next task: $taskIndex of $totalTasks

Review the changes, then run /ralph-specum:implement to continue.
```

### Available Context at Pause Time

| Source | What's Available | How to Access |
|--------|-----------------|---------------|
| `.ralph-state.json` | taskIndex, totalTasks, phase | Read state file |
| `tasks.md` | Full task list with descriptions | Parse task file |
| `.progress.md` | Completed tasks, learnings | Read progress |
| `design.md` | Architecture diagram, component relationships | Read design |
| Current task block | Do, Files, Done when, Commit | Already parsed by coordinator |
| Next task block | Do, Files, description | Parse next from tasks.md |

### Design.md Architecture Section

Design files often contain ASCII diagrams already:
```markdown
## Architecture

+-------------+     +--------------+
| Coordinator | --> | spec-executor|
+-------------+     +--------------+
```

This existing pattern can be leveraged/extended.

### Integration Point

Modify the pause message output in implement.md coordinator prompt, section 8 "reviewEachTask Pause Check":

**Current** (lines 448-456):
```
2. Output pause message:
   ```
   TASK COMPLETE - PAUSED FOR REVIEW
   ...
   ```
```

**Proposed**: Add ASCII diagram after task info, before resume instructions.

### Diagram Generation Strategy

Claude (the coordinator) can generate ASCII diagrams dynamically:

1. **Read design.md** for component names and relationships
2. **Read tasks.md** for phase structure and task grouping
3. **Generate diagram** showing:
   - Components affected by completed task (marked)
   - Components to be modified by next task (highlighted)
   - Overall progress through phases

Example output format:
```
TASK COMPLETE - PAUSED FOR REVIEW

Task 1.3 completed: Add config storage to start.md
Next task: 2.1 Modify spec-executor for autoCommit

## Architecture State

  Phase 1 (POC)           Phase 2 (Integration)
  +--------------+        +-----------------+
  | 1.1 Schema   | [DONE] | 2.1 spec-exec   | <-- NEXT
  | 1.2 Questions| [DONE] | 2.2 Layer 2     |
  | 1.3 Storage  | [DONE] | 2.3 reviewTask  |
  | 1.4 Progress | [DONE] | 2.4 autoPush    |
  +--------------+        +-----------------+

  Files modified this task:    Files for next task:
  - start.md                   - spec-executor.md

Review the changes, then run /ralph-specum:implement to continue.
```

## Feasibility Assessment

| Aspect | Assessment | Notes |
|--------|------------|-------|
| Technical Viability | High | Pure prompt modification, no new tools |
| Effort Estimate | S | ~4-6 tasks, markdown-only changes |
| Risk Level | Low | Additive feature, doesn't break existing flow |

## Related Specs

| Spec | Relevance | Reason | May Need Update |
|------|-----------|--------|-----------------|
| config-questions | High | Implements reviewEachTask config | No - this uses its output |
| implement-ralph-wiggum | Medium | Defines coordinator prompt structure | Possibly - pause message location |
| goal-interview | Low | Unrelated feature | No |

## Quality Commands

| Type | Command | Source |
|------|---------|--------|
| Lint | Not found | No package.json in repo root |
| TypeCheck | Not found | No build infrastructure |
| Test | Not found | Plugin is markdown-only |
| Build | Not found | No build step |

**Local CI**: Plugin is pure markdown. Verification via grep patterns and structure checks.

## Recommendations for Requirements

1. **Modify implement.md coordinator pause message**
   - Add diagram generation section after "Task X completed"
   - Before "Review the changes" instruction

2. **Define diagram content**:
   - Phase progress (which phase, tasks completed/remaining)
   - Component state (from design.md or inferred from Files sections)
   - File delta (files changed vs files coming next)

3. **Keep diagrams simple**
   - Use ASCII-only characters (`+`, `-`, `|`, `>`) for compatibility
   - Max width ~60 chars for terminal readability
   - Focus on "where we are, where we're going"

4. **Optional: Read design.md for richer context**
   - If design.md has Architecture section, reference it
   - If not, generate from task Files sections

5. **No external tools required**
   - Claude generates diagrams inline
   - No dependencies to install

## Open Questions

1. **How detailed should diagrams be?** Options:
   - Minimal: Just task progress bar
   - Medium: Phase overview + next task
   - Full: Component diagram with file relationships

2. **Should diagram style be configurable?** Could add to config questions, but adds complexity.

3. **What if design.md has no architecture section?** Fall back to task-based diagram only.

## Sources

- `/home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md` - Coordinator prompt, pause logic
- `/home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/start.md` - Config questions (reviewEachTask)
- `/home/omerr/repos/smart-ralph/specs/config-questions/.progress.md` - reviewEachTask implementation details
- `/home/omerr/repos/smart-ralph/specs/config-questions/design.md` - Design patterns for config
- [Box-drawing characters - Wikipedia](https://en.wikipedia.org/wiki/Box-drawing_characters)
- [ASCII diagrams gist](https://gist.github.com/dsample/79a97f38bf956f37a0f99ace9df367b9)
- [Baeldung - ASCII diagrams in shell](https://www.baeldung.com/linux/shell-ascii-diagrams)
