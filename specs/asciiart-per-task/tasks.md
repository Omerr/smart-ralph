# Tasks: ASCII Art Per-Task Diagrams

## Phase 1: Make It Work (POC)

Focus: Add diagram generation instructions to implement.md coordinator prompt.

- [x] 1.1 Add diagram generation instructions to reviewEachTask pause block
  - **Do**:
    1. Open `plugins/ralph-specum/commands/implement.md`
    2. Locate section 8 "State Update", reviewEachTask Pause Check (lines 425-460)
    3. Insert new **Generate Progress Diagram** subsection after awaitingApproval flag write (step 1) and before pause message output (step 2)
    4. Add diagram generation instructions per design.md specification:
       - Parse tasks.md for phase headers and task list
       - Determine visible phases (current + next)
       - Render horizontal phase columns with task boxes
       - Mark completed tasks with [x], next task with `<-- NEXT`
       - Include constraints: 70 char max, 12 char task labels
  - **Files**: `plugins/ralph-specum/commands/implement.md`
  - **Done when**: Diagram generation instructions present between step 1 and step 2
  - **Verify**: `grep -n "Progress Diagram" /home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md`
  - **Commit**: `feat(implement): add ASCII progress diagram generation instructions`
  - _Requirements: FR-1, FR-2, FR-3, FR-9, AC-1.1_
  - _Design: Coordinator Prompt Modifications_

- [x] 1.2 Update pause message template to include diagram placeholder
  - **Do**:
    1. In same file, locate the pause message output template (current lines 449-456)
    2. Modify template to include `## Progress` section between task info and resume instructions
    3. Add placeholder showing where generated diagram goes
    4. Ensure diagram appears after "Next task: X of Y" and before "Review the changes..."
  - **Files**: `plugins/ralph-specum/commands/implement.md`
  - **Done when**: Pause message template includes `## Progress` section with diagram placeholder
  - **Verify**: `grep -A5 "PAUSED FOR REVIEW" /home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md | grep -q "Progress"`
  - **Commit**: `feat(implement): add progress diagram placeholder to pause message`
  - _Requirements: FR-1, AC-1.4_
  - _Design: Modified Pause Output Template_

- [x] 1.3 Add edge case handling instructions
  - **Do**:
    1. In diagram generation section, add handling for edge cases:
       - First task: show "Starting spec" context, no previous phase summary
       - Last task of spec: show "Final task!" in summary, no NEXT marker
       - Single-task spec: simplified message "Single task spec - completing now"
       - Single-task phase: render phase box with single task
       - Long task names: truncate to 12 chars + "..."
       - Many tasks in phase (>6): show first 3, "...", last task
  - **Files**: `plugins/ralph-specum/commands/implement.md`
  - **Done when**: All edge cases documented in diagram generation instructions
  - **Verify**: `grep -c "First task\|Last task\|Single-task\|truncate" /home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md | grep -q "[3-9]"`
  - **Commit**: `feat(implement): add edge case handling for progress diagrams`
  - _Requirements: FR-8, AC-4.1, AC-4.2, AC-4.3, AC-4.4_
  - _Design: Edge Case Handling_

- [x] 1.4 Add next task files extraction instructions
  - **Do**:
    1. In diagram generation section, add instructions to extract Files from next task block
    2. Include in diagram output as "Files: <comma-separated list>"
    3. Limit file list to avoid overwhelming output (show first 3 + count if more)
  - **Files**: `plugins/ralph-specum/commands/implement.md`
  - **Done when**: Files extraction instructions present in diagram generation section
  - **Verify**: `grep -n "Files:" /home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md | grep -q "next task"`
  - **Commit**: `feat(implement): add next task files to progress diagram`
  - _Requirements: FR-6, AC-3.2, AC-3.3_
  - _Design: Prompt Addition step 3_

- [x] 1.5 POC Checkpoint
  - **Do**: Verify all diagram generation instructions are properly integrated
  - **Done when**: All grep patterns pass, instructions logically flow
  - **Verify**: `grep -c "Progress Diagram\|Phase N\|<-- NEXT\|First task\|Files:" /home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md | grep -q "[5-9]"`
  - **Commit**: `feat(asciiart): complete POC for progress diagrams`

## Phase 2: Refactoring

After POC validated, clean up and ensure consistency.

- [ ] 2.1 Ensure diagram format consistency with design spec
  - **Do**:
    1. Review diagram template against design.md specification
    2. Verify all markers match: `[x]`, `<-- NEXT`, `+`, `-`, `|`
    3. Ensure 70 char max width constraint is documented
    4. Verify phase column layout matches horizontal design
  - **Files**: `plugins/ralph-specum/commands/implement.md`
  - **Done when**: Diagram format matches design.md exactly
  - **Verify**: `grep -c "\[x\]\|<-- NEXT\|70 char" /home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md | grep -q "[3-9]"`
  - **Commit**: `refactor(implement): align diagram format with design spec`
  - _Design: Diagram Format Specification, Layout Rules_

- [ ] 2.2 [VERIFY] Quality checkpoint: structure validation
  - **Do**: Validate implement.md structure is intact after modifications
  - **Verify**: `grep -c "^### [0-9]" /home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md | grep -q "10"`
  - **Done when**: All 10 sections still present (1-10)
  - **Commit**: `chore(implement): pass quality checkpoint` (only if fixes needed)

## Phase 3: Testing

Skipped per interview - minimal testing depth, markdown-only changes.

## Phase 4: Quality Gates

- [ ] 4.1 Version bump in plugin.json
  - **Do**:
    1. Open `plugins/ralph-specum/.claude-plugin/plugin.json`
    2. Bump version from "2.5.8" to "2.5.9"
  - **Files**: `plugins/ralph-specum/.claude-plugin/plugin.json`
  - **Done when**: Version is "2.5.9"
  - **Verify**: `grep '"version": "2.5.9"' /home/omerr/repos/smart-ralph/plugins/ralph-specum/.claude-plugin/plugin.json`
  - **Commit**: `chore(ralph-specum): bump version to 2.5.9`

- [ ] 4.2 Version bump in marketplace.json
  - **Do**:
    1. Open `.claude-plugin/marketplace.json`
    2. Update ralph-specum version from "2.5.8" to "2.5.9"
  - **Files**: `.claude-plugin/marketplace.json`
  - **Done when**: ralph-specum version is "2.5.9"
  - **Verify**: `grep -A2 '"name": "ralph-specum"' /home/omerr/repos/smart-ralph/.claude-plugin/marketplace.json | grep -q '"version": "2.5.9"'`
  - **Commit**: `chore(marketplace): bump ralph-specum to 2.5.9`

- [ ] 4.3 [VERIFY] Final validation: all diagram instructions present
  - **Do**: Run comprehensive grep to verify all key elements present
  - **Verify**: `grep -E "Progress Diagram|Phase N|<-- NEXT|First task|Last task|Single-task|truncate|Files:" /home/omerr/repos/smart-ralph/plugins/ralph-specum/commands/implement.md | wc -l | grep -q "[8-9][0-9]*\|[1-9][0-9]"`
  - **Done when**: At least 8 key patterns found in implement.md
  - **Commit**: None

- [ ] 4.4 Create PR and verify
  - **Do**:
    1. Verify current branch is feature branch: `git branch --show-current`
    2. If on default branch, STOP and alert user
    3. Push branch: `git push -u origin <branch-name>`
    4. Create PR: `gh pr create --title "feat(implement): add ASCII progress diagrams per task" --body "..."`
  - **Verify**: `gh pr checks` shows all green (or no CI configured)
  - **Done when**: PR created and ready for review
  - **Commit**: None (PR creation, not commit)

## Notes

- **POC shortcuts taken**: None significant - this is markdown-only changes
- **Production TODOs**:
  - Consider making diagram style configurable in future
  - Consider extracting design.md architecture diagram in future enhancement
- **Testing**: Skipped per interview - verification via grep patterns
- **Deployment**: Plugin changes take effect on Claude Code restart
