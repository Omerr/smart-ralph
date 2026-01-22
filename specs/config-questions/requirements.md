---
spec: config-questions
phase: requirements
created: 2026-01-22
---

# Requirements: Config Questions

## Goal

Add configuration questions to `/ralph-specum:start` allowing users to control: auto-commit behavior, task review pauses, and auto-push/PR creation.

## User Stories

### US-1: Configure Auto-Commit Behavior

**As a** developer using Ralph
**I want to** choose whether Ralph commits automatically after each task
**So that** I can review changes before they're committed (if disabled)

**Acceptance Criteria:**
- [ ] AC-1.1: Question "Should Ralph commit automatically after each task?" appears during start
- [ ] AC-1.2: User can select Yes (default) or No
- [ ] AC-1.3: Selection stored in `.ralph-state.json` config.autoCommit
- [ ] AC-1.4: When autoCommit=false, spec-executor stages files but skips commit
- [ ] AC-1.5: When autoCommit=false, Layer 2 verification (uncommitted files check) is skipped

### US-2: Configure Task Review Pauses

**As a** developer using Ralph
**I want to** choose whether Ralph pauses after each task for manual review
**So that** I can inspect and validate changes before the next task starts

**Acceptance Criteria:**
- [ ] AC-2.1: Question "Should Ralph pause for review after each task?" appears during start
- [ ] AC-2.2: User can select Yes or No (default)
- [ ] AC-2.3: Selection stored in `.ralph-state.json` config.reviewEachTask
- [ ] AC-2.4: When reviewEachTask=true, coordinator sets awaitingApproval=true after task completion
- [ ] AC-2.5: User must run `/ralph-specum:implement` to resume after pause

### US-3: Configure Auto-Push and PR Creation

**As a** developer using Ralph
**I want to** choose whether Ralph pushes and creates a PR automatically
**So that** I can handle git remote operations myself if preferred

**Acceptance Criteria:**
- [ ] AC-3.1: Question "Should Ralph push and create PR when complete?" appears during start
- [ ] AC-3.2: User can select Yes (default) or No
- [ ] AC-3.3: Selection stored in `.ralph-state.json` config.autoPushAndPR
- [ ] AC-3.4: When autoPushAndPR=false, task 4.2 (PR creation) marked as manual step
- [ ] AC-3.5: User informed they need to push and create PR manually

### US-4: Skip Config in Quick Mode

**As a** developer using quick mode
**I want to** skip configuration questions
**So that** quick mode remains non-interactive

**Acceptance Criteria:**
- [ ] AC-4.1: Config questions not asked when --quick flag present
- [ ] AC-4.2: If global config exists, use global defaults in quick mode
- [ ] AC-4.3: If no global config, use built-in defaults: autoCommit=true, reviewEachTask=false, autoPushAndPR=true

### US-5: Display Config in Progress

**As a** developer reviewing spec progress
**I want to** see current config settings in .progress.md
**So that** I know how the spec is configured

**Acceptance Criteria:**
- [ ] AC-5.1: Configuration section in .progress.md shows all 3 settings
- [ ] AC-5.2: Values reflect user's actual choices

### US-6: Global Config Defaults

**As a** developer starting multiple specs
**I want to** store my config preferences globally
**So that** I don't have to re-enter the same choices every time

**Acceptance Criteria:**
- [ ] AC-6.1: Global config stored at `~/.config/ralph-specum/config.json`
- [ ] AC-6.2: On start, if global config exists, show saved defaults to user
- [ ] AC-6.3: User asked "Use saved defaults? (Yes/No/Customize)"
- [ ] AC-6.4: If Yes, config loaded from global file without further questions
- [ ] AC-6.5: If No, standard config questions asked (US-1 to US-3)
- [ ] AC-6.6: If Customize, standard config questions asked with global values as defaults
- [ ] AC-6.7: After customization, user asked "Save as new defaults?"
- [ ] AC-6.8: If save accepted, global config file updated
- [ ] AC-6.9: First-time users (no global config) go directly to config questions
- [ ] AC-6.10: Global config file created after first run if user opts to save

## Functional Requirements

| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|---------------------|
| FR-1 | Add config questions to start.md after branch questions, before goal interview | High | Questions appear in correct order |
| FR-2 | Use AskUserQuestion tool with 3 questions in single call | High | All 3 questions asked together |
| FR-3 | Store responses in .ralph-state.json config object | High | Config persists across sessions |
| FR-4 | Modify spec-executor to check autoCommit before committing | High | Commits skipped when false |
| FR-5 | Modify implement.md Layer 2 to skip check when autoCommit=false | High | No false failures on uncommitted files |
| FR-6 | Modify coordinator to check reviewEachTask after TASK_COMPLETE | High | Pauses when true |
| FR-7 | Pass autoPushAndPR to task-planner for Phase 4 task generation | Medium | Manual task generated when false |
| FR-8 | Update .progress.md Configuration section in start.md | Medium | Config visible to user |
| FR-9 | Use sensible defaults in quick mode | Medium | autoCommit=true, reviewEachTask=false, autoPushAndPR=true |
| FR-10 | Store global config at ~/.config/ralph-specum/config.json | High | File created/updated correctly |
| FR-11 | Check for global config existence on start | High | Detection works for new and returning users |
| FR-12 | Display saved defaults with 3-option prompt (Use/No/Customize) | High | Clear user choice flow |
| FR-13 | Pre-fill config questions with global defaults when customizing | Medium | Values shown as defaults in prompts |
| FR-14 | Prompt to save customized config as new defaults | Medium | Only asked after customization |
| FR-15 | Create ~/.config/ralph-specum/ directory if not exists | High | No errors on first run |
| FR-16 | Quick mode uses global defaults if exist, else built-in defaults | Medium | No prompts in quick mode |

## Non-Functional Requirements

| ID | Requirement | Metric | Target |
|----|-------------|--------|--------|
| NFR-1 | Config questions should not slow down start | Question count | Single AskUserQuestion call |
| NFR-2 | Config changes must not break existing specs | Backward compat | Existing specs continue working with defaults |

## Glossary

- **autoCommit**: Config option controlling automatic git commits after each task
- **reviewEachTask**: Config option to pause execution for user review between tasks
- **autoPushAndPR**: Config option controlling automatic push and PR creation at end
- **Layer 2 verification**: Coordinator check that spec files are committed before advancing taskIndex
- **quick mode**: Non-interactive mode using --quick flag that skips all prompts
- **global config**: User-level config stored at ~/.config/ralph-specum/config.json, persists across all specs
- **per-spec config**: Config stored in .ralph-state.json, specific to one spec run

## Out of Scope

- Mid-spec config editing (no `/ralph-specum:config` command)
- Per-task config overrides
- Undo/redo for config choices
- Migration tool for existing specs to global config

## Dependencies

- Existing AskUserQuestion tool availability in start.md (confirmed)
- Existing config object structure in .ralph-state.json (confirmed)
- spec-executor, implement.md, task-planner all need updates
- File system access to ~/.config/ directory
- Node.js fs module or shell commands for global config file operations

## Success Criteria

- Config questions appear when running `/ralph-specum:start` in normal mode
- Config questions do not appear in quick mode
- All 3 config options correctly control their respective behaviors
- Existing specs without config continue working (backward compatible defaults)
- Global config persists across spec sessions
- Returning users see "Use saved defaults?" prompt
- New users go through full config flow

## Unresolved Questions

1. **Layer 2 conflict with autoCommit=false**: When autoCommit disabled, uncommitted files check fails. Solution in requirements: skip Layer 2 when autoCommit=false. Alternative: require user to commit before next task runs. Recommend: skip Layer 2 (simpler).

2. **reviewEachTask interaction with parallel tasks**: If parallel batch running and reviewEachTask=true, should pause happen after entire batch or after each parallel task? Recommend: pause after entire batch completes (cleaner UX).

3. **Global config schema versioning**: Should global config include a version field for future migration? Recommend: yes, add `"version": 1` to support future schema changes.

4. **XDG Base Directory compliance**: Using ~/.config/ralph-specum/ follows XDG spec on Linux. Should fallback to ~/.ralph-specum/ on systems without XDG support? Recommend: use ~/.config/ universally (works on macOS too).

## Next Steps

1. Review requirements and provide feedback
2. Run `/ralph-specum:design` to generate technical design
