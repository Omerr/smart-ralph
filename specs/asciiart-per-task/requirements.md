# Requirements: ASCII Art Per-Task Diagrams

## Goal

Display ASCII art progress diagrams after each task when `reviewEachTask=true`, showing current architecture state and next task direction.

## User Stories

### US-1: See Progress Diagram After Task Completion
**As a** developer reviewing task-by-task execution
**I want to** see an ASCII diagram showing my progress through phases
**So that** I understand where I am and what comes next without reading task files

**Acceptance Criteria:**
- [ ] AC-1.1: Diagram appears after "Task X completed" message when reviewEachTask=true
- [ ] AC-1.2: Diagram shows completed tasks marked differently from pending tasks
- [ ] AC-1.3: Diagram highlights current (just completed) and next task
- [ ] AC-1.4: Diagram displays before "Review the changes..." instruction

### US-2: Understand Phase Progress
**As a** developer managing multi-phase specs
**I want to** see which phase I'm in and progress within that phase
**So that** I can estimate remaining work and understand feature scope

**Acceptance Criteria:**
- [ ] AC-2.1: Phase name/number displayed in diagram
- [ ] AC-2.2: Tasks grouped by phase visually
- [ ] AC-2.3: Completed vs remaining tasks clearly differentiated
- [ ] AC-2.4: Works with POC-first 4-phase workflow (Make It Work, Refactoring, Testing, Quality Gates)

### US-3: See File Context for Next Task
**As a** developer preparing for next task
**I want to** see which files will be modified next
**So that** I can review relevant code before resuming

**Acceptance Criteria:**
- [ ] AC-3.1: Files from completed task's "Files" section shown (optional)
- [ ] AC-3.2: Files from next task's "Files" section highlighted
- [ ] AC-3.3: File list limited to avoid overwhelming output

### US-4: Handle Edge Cases Gracefully
**As a** developer on first or last task
**I want to** see appropriate diagrams without errors
**So that** the feature works throughout entire spec lifecycle

**Acceptance Criteria:**
- [ ] AC-4.1: First task shows "Starting" context, no "previous phase" info
- [ ] AC-4.2: Last task of phase shows transition to next phase
- [ ] AC-4.3: Single-task phases display correctly
- [ ] AC-4.4: Spec with only 1 total task shows simplified diagram

## Functional Requirements

| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|---------------------|
| FR-1 | Generate ASCII diagram in coordinator pause message | High | Diagram appears inline in pause output |
| FR-2 | Read tasks.md for phase structure and task list | High | Parse phase headers and task items |
| FR-3 | Show phase-based progress visualization | High | Phases as columns/boxes, tasks as rows |
| FR-4 | Mark completed tasks with [DONE] or checkmark | High | Visual distinction from pending tasks |
| FR-5 | Highlight next task with arrow or marker | High | Clear indicator like `<-- NEXT` |
| FR-6 | Extract file list from current/next task blocks | Medium | Show Files section content |
| FR-7 | Use terminal-compatible ASCII characters | High | Only `+`, `-`, `|`, `>`, `[`, `]` |
| FR-8 | Limit diagram width to 70 characters | Medium | Readable in standard terminals |
| FR-9 | Generate diagram dynamically from context | High | No external tools or dependencies |

## Non-Functional Requirements

| ID | Requirement | Metric | Target |
|----|-------------|--------|--------|
| NFR-1 | Terminal compatibility | Character set | ASCII-only (`+-|>[]`) |
| NFR-2 | Output width | Max chars | 70 characters max |
| NFR-3 | Diagram generation | Dependency | No external tools |
| NFR-4 | Performance | Impact | Negligible (inline generation) |

## Diagram Specification

### Standard Format

```
## Progress Diagram

  Phase 1 (POC)           Phase 2 (Integration)
  +--------------+        +-----------------+
  | 1.1 Schema   | [DONE] | 2.1 spec-exec   | <-- NEXT
  | 1.2 Questions| [DONE] | 2.2 Layer 2     |
  | 1.3 Storage  | [DONE] | 2.3 reviewTask  |
  +--------------+        +-----------------+

  Completed: 4/8 tasks | Current phase: Integration

  Next task files: spec-executor.md
```

### Simplified Format (small specs)

```
## Progress Diagram

  [ ] --> [x] --> [x] --> [ ] --> [ ]
            ^             |
          Done          Next

  Completed: 2/5 tasks
```

## Glossary

- **reviewEachTask**: Config option that pauses execution after each task for user review
- **Phase**: Grouping of related tasks (POC, Refactoring, Testing, Quality Gates)
- **Pause message**: Output displayed when execution pauses for review

## Out of Scope

- Configurable diagram styles (always use standard format)
- Interactive diagram manipulation
- Diagram persistence to file
- Architecture diagrams from design.md (may enhance later)
- Color/ANSI codes in output
- Unicode box-drawing characters

## Dependencies

- `reviewEachTask` config from config-questions spec (already implemented)
- tasks.md with phase structure (standard format)
- implement.md coordinator prompt (integration point)

## Unresolved Questions

1. **How many tasks to show per phase?** Recommend: all tasks in current and adjacent phases, collapse distant phases to counts
2. **Include design.md architecture diagram?** Recommend: defer to future enhancement, keep scope minimal

## Success Criteria

- ASCII diagram appears after every task pause when reviewEachTask=true
- Diagram correctly reflects task completion state
- Diagram readable in standard 80-char terminal
- No errors on first task, last task, or single-task specs
- Zero external dependencies added

## Next Steps

1. Run /ralph-specum:design to create technical design
2. Design will specify exact coordinator prompt modifications
3. Tasks will implement changes to implement.md section 8
