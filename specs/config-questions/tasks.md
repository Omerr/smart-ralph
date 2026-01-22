---
spec: config-questions
phase: tasks
total_tasks: 12
created: 2026-01-22
---

# Tasks: Config Questions

## Phase 1: Make It Work (POC)

Focus: Add config questions to start.md and wire them through to state. Skip schema validation.

- [ ] 1.1 Add config schema to spec.schema.json
  - **Do**:
    1. Open `plugins/ralph-specum/schemas/spec.schema.json`
    2. Add `config` object to state definition properties (after `taskResults`)
    3. Define config schema with `autoCommit`, `reviewEachTask`, `autoPushAndPR` boolean fields
    4. Add version field for global config migration support
  - **Files**: `plugins/ralph-specum/schemas/spec.schema.json`
  - **Done when**: Schema includes config object with all 3 boolean fields plus version
  - **Verify**: `cat plugins/ralph-specum/schemas/spec.schema.json | grep -A 10 '"config"'`
  - **Commit**: `feat(config): add config object to state schema`
  - _Requirements: FR-3, AC-1.3, AC-2.3, AC-3.3_
  - _Design: Config Storage (per-spec)_

- [ ] 1.2 Add config questions section to start.md
  - **Do**:
    1. Open `plugins/ralph-specum/commands/start.md`
    2. After "Branch Management" section (line ~97), add new section "## Config Questions (Pre-Interview)"
    3. Add quick mode check to skip questions when `--quick` in $ARGUMENTS
    4. Add global config existence check using `cat ~/.config/ralph-specum/config.json`
    5. Add 3-way prompt for returning users: "Use saved defaults? (Yes/No/Customize)"
    6. Add AskUserQuestion for 3 config questions with Yes/No options
    7. Add save defaults prompt after config questions answered
  - **Files**: `plugins/ralph-specum/commands/start.md`
  - **Done when**: Config section exists between branch and goal interview sections
  - **Verify**: `grep -n "Config Questions" plugins/ralph-specum/commands/start.md && grep -n "autoCommit\|reviewEachTask\|autoPushAndPR" plugins/ralph-specum/commands/start.md | head -5`
  - **Commit**: `feat(config): add config questions section to start.md`
  - _Requirements: FR-1, FR-2, AC-1.1, AC-1.2, AC-2.1, AC-2.2, AC-3.1, AC-3.2_
  - _Design: Config Question Flow (in start.md)_

- [ ] 1.3 Add global config read/write logic to start.md
  - **Do**:
    1. In start.md config section, add bash commands for:
       - Reading global config: `cat ~/.config/ralph-specum/config.json 2>/dev/null`
       - Creating directory: `mkdir -p ~/.config/ralph-specum`
       - Writing config: `cat > ~/.config/ralph-specum/config.json << EOF`
    2. Add logic for quick mode to use global defaults if exist, else built-in defaults
    3. Add version: 1 to global config format
  - **Files**: `plugins/ralph-specum/commands/start.md`
  - **Done when**: start.md includes commands for global config file operations
  - **Verify**: `grep -n "~/.config/ralph-specum" plugins/ralph-specum/commands/start.md | head -5`
  - **Commit**: `feat(config): add global config read/write logic`
  - _Requirements: FR-10, FR-11, FR-15, FR-16, AC-6.1, AC-6.2_
  - _Design: Global Config Manager_

- [ ] 1.4 Add per-spec config storage and progress update to start.md
  - **Do**:
    1. Update start.md state initialization to include config object
    2. Add logic to write config values to .ralph-state.json
    3. Add Configuration section to .progress.md template showing all 3 settings
  - **Files**: `plugins/ralph-specum/commands/start.md`
  - **Done when**: State init includes config, progress shows config section
  - **Verify**: `grep -n '"config"' plugins/ralph-specum/commands/start.md && grep -n "## Configuration" plugins/ralph-specum/commands/start.md`
  - **Commit**: `feat(config): add per-spec config storage and progress display`
  - _Requirements: FR-3, FR-8, AC-5.1, AC-5.2_
  - _Design: Config Storage (per-spec), Update .progress.md Configuration Section_

## Phase 2: Integration

Wire config options to their respective integration points.

- [ ] 2.1 Modify spec-executor for conditional commits (autoCommit)
  - **Do**:
    1. Open `plugins/ralph-specum/agents/spec-executor.md`
    2. In "Commit Discipline" section (around line 266), add conditional logic
    3. Add bash command to read autoCommit: `jq -r '.config.autoCommit // true' ./specs/<spec>/.ralph-state.json`
    4. If autoCommit=true (or missing): commit as usual
    5. If autoCommit=false: stage files only, skip commit, log to progress
    6. Document that TASK_COMPLETE still outputs even with autoCommit=false
  - **Files**: `plugins/ralph-specum/agents/spec-executor.md`
  - **Done when**: Commit section has conditional logic checking autoCommit
  - **Verify**: `grep -n "autoCommit" plugins/ralph-specum/agents/spec-executor.md | head -5`
  - **Commit**: `feat(config): add conditional commit logic to spec-executor`
  - _Requirements: FR-4, AC-1.4_
  - _Design: spec-executor.md Changes_

- [ ] 2.2 Modify implement.md Layer 2 to skip when autoCommit=false
  - **Do**:
    1. Open `plugins/ralph-specum/commands/implement.md`
    2. Find Layer 2 section "Uncommitted Spec Files Check" (around line 323)
    3. Add autoCommit config read before Layer 2 check
    4. Add conditional: "If autoCommit=false: Skip Layer 2 entirely. Proceed to Layer 3."
    5. Keep existing Layer 2 logic for autoCommit=true case
  - **Files**: `plugins/ralph-specum/commands/implement.md`
  - **Done when**: Layer 2 has conditional skip based on autoCommit config
  - **Verify**: `grep -n "autoCommit" plugins/ralph-specum/commands/implement.md | head -5`
  - **Commit**: `feat(config): skip Layer 2 verification when autoCommit disabled`
  - _Requirements: FR-5, AC-1.5_
  - _Design: implement.md Coordinator Changes - Location 1_

- [ ] 2.3 Modify implement.md for reviewEachTask pause behavior
  - **Do**:
    1. In implement.md, find State Update section (section 8, around line 375)
    2. Add reviewEachTask config read: `jq -r '.config.reviewEachTask // false'`
    3. Add conditional after task completion:
       - If reviewEachTask=true AND taskIndex < totalTasks: set awaitingApproval=true, output pause message, STOP
       - If reviewEachTask=false: continue to next task
  - **Files**: `plugins/ralph-specum/commands/implement.md`
  - **Done when**: State Update section checks reviewEachTask and pauses when true
  - **Verify**: `grep -n "reviewEachTask" plugins/ralph-specum/commands/implement.md | head -3`
  - **Commit**: `feat(config): add reviewEachTask pause behavior`
  - _Requirements: FR-6, AC-2.4, AC-2.5_
  - _Design: implement.md Coordinator Changes - Location 2_

- [ ] 2.4 Modify task-planner for conditional PR task (autoPushAndPR)
  - **Do**:
    1. Open `plugins/ralph-specum/agents/task-planner.md`
    2. Find Phase 4 task 4.2 section (around line 378)
    3. Add conditional task generation based on autoPushAndPR config
    4. If autoPushAndPR=true (default): keep normal 4.2 task
    5. If autoPushAndPR=false: generate manual notification task instead
    6. Add instruction to read config from .progress.md Configuration section
  - **Files**: `plugins/ralph-specum/agents/task-planner.md`
  - **Done when**: Phase 4 section has conditional 4.2 task generation
  - **Verify**: `grep -n "autoPushAndPR" plugins/ralph-specum/agents/task-planner.md | head -3`
  - **Commit**: `feat(config): add conditional PR task generation`
  - _Requirements: FR-7, AC-3.4, AC-3.5_
  - _Design: task-planner.md Changes_

## Phase 3: Testing

Skip - Interview indicated minimal/POC only (markdown-only changes to plugin definition files).

## Phase 4: Quality Gates

- [ ] 4.1 [VERIFY] Verify all config options documented in files
  - **Do**:
    1. Check start.md has config questions for all 3 options
    2. Check spec-executor.md handles autoCommit
    3. Check implement.md handles autoCommit (Layer 2) and reviewEachTask
    4. Check task-planner.md handles autoPushAndPR
    5. Check schema has config object
  - **Verify**: `grep -l "autoCommit" plugins/ralph-specum/commands/start.md plugins/ralph-specum/agents/spec-executor.md plugins/ralph-specum/commands/implement.md && grep -l "reviewEachTask" plugins/ralph-specum/commands/start.md plugins/ralph-specum/commands/implement.md && grep -l "autoPushAndPR" plugins/ralph-specum/commands/start.md plugins/ralph-specum/agents/task-planner.md`
  - **Done when**: All config options appear in their expected files
  - **Commit**: None (verification only)

- [ ] 4.2 [VERIFY] AC checklist verification
  - **Do**:
    1. Read requirements.md for all AC-* criteria
    2. Verify AC-1.* (autoCommit): start.md questions, state storage, spec-executor conditional
    3. Verify AC-2.* (reviewEachTask): start.md questions, state storage, implement.md pause
    4. Verify AC-3.* (autoPushAndPR): start.md questions, state storage, task-planner conditional
    5. Verify AC-4.* (quick mode): start.md quick mode check
    6. Verify AC-5.* (progress display): start.md Configuration section
    7. Verify AC-6.* (global config): start.md global config logic
  - **Verify**: Run grep for each AC requirement pattern in appropriate files:
    - `grep -q "Should Ralph commit automatically" plugins/ralph-specum/commands/start.md && echo "AC-1.1 PASS"`
    - `grep -q "config.autoCommit" plugins/ralph-specum/agents/spec-executor.md && echo "AC-1.4 PASS"`
    - `grep -q "Skip Layer 2" plugins/ralph-specum/commands/implement.md && echo "AC-1.5 PASS"`
    - `grep -q "reviewEachTask" plugins/ralph-specum/commands/implement.md && echo "AC-2.4 PASS"`
    - `grep -q "--quick" plugins/ralph-specum/commands/start.md && echo "AC-4.1 PASS"`
    - `grep -q "## Configuration" plugins/ralph-specum/commands/start.md && echo "AC-5.1 PASS"`
    - `grep -q "~/.config/ralph-specum" plugins/ralph-specum/commands/start.md && echo "AC-6.1 PASS"`
  - **Done when**: All AC requirements verified present in files
  - **Commit**: None (verification only)

- [ ] 4.3 Version bump for plugin
  - **Do**:
    1. Read current version from `plugins/ralph-specum/.claude-plugin/plugin.json`
    2. Increment patch version (e.g., 0.4.0 -> 0.4.1)
    3. Update version in `plugins/ralph-specum/.claude-plugin/plugin.json`
    4. Update same version in `.claude-plugin/marketplace.json`
  - **Files**: `plugins/ralph-specum/.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`
  - **Done when**: Both files have matching incremented version
  - **Verify**: `jq -r '.version' plugins/ralph-specum/.claude-plugin/plugin.json && jq -r '.plugins[] | select(.name == "ralph-specum") | .version' .claude-plugin/marketplace.json`
  - **Commit**: `chore(config): bump version for config questions feature`
  - _Requirements: Version bump required per CLAUDE.md_

- [ ] 4.4 Create PR and verify CI
  - **Do**:
    1. Verify current branch is feature branch: `git branch --show-current`
    2. Push branch: `git push -u origin $(git branch --show-current)`
    3. Create PR: `gh pr create --title "feat(config): add config questions to /ralph-specum:start" --body "Adds configuration prompts for autoCommit, reviewEachTask, and autoPushAndPR options"`
  - **Verify**: `gh pr checks` shows all green (or `gh pr view --json state -q .state` returns OPEN)
  - **Done when**: PR created, CI passes
  - **Commit**: None (PR creation only)
  - _Requirements: Phase 4 deliverable is PR with passing CI_

## Notes

- **POC shortcuts taken**: Schema validation not enforced at runtime, global config error handling minimal
- **Production TODOs**: Add config command for mid-spec editing, add migration for schema version changes
- **Testing skipped**: Per interview - this is markdown-only plugin definition changes
- **Backward compat**: All config values default to current behavior (true, false, true) when missing
