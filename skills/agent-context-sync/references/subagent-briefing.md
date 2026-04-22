# Subagent Briefing

Use this reference before dispatching a subagent or integrating subagent results.

## Purpose

A subagent brief gives a subagent the minimum project context needed to complete a bounded mission without re-reading the whole repository or inheriting noisy memory.

## When To Create A Brief

Create a brief when:

- the subagent role matters
- prior decisions constrain the work
- the repository has enough files that blind scanning is wasteful
- the task should avoid reopened rejected alternatives
- the expected output needs a specific format

Do not create a brief for trivial one-file checks unless the user asks.

## Required Sections

Each brief must include:

- mission
- role
- required context
- relevant decisions
- files to read
- files not to read unless needed
- constraints
- expected output
- non-goals
- expiry

Use `assets/subagent-brief-template.md`.

## Scoping Rules

- Give one mission per brief.
- Prefer decision IDs and file paths over copied background.
- List the first files to read.
- List files or topics that should not be reopened unless needed.
- Include output format and level of detail.
- Include write boundaries if the subagent may edit files.
- Do not include platform memory unless it is necessary and user-approved.

## Role Examples

- `supporter`: strengthen the current design.
- `light skeptic`: find moderate risks and missing assumptions.
- `strong skeptic`: challenge whether the design should exist.
- `explorer`: answer a narrow codebase or research question.
- `worker`: implement a bounded file set.
- `reviewer`: inspect outputs for bugs, gaps, or requirement misses.

## Integrating Results

After the subagent returns:

1. Summarize useful findings in `session-log.md`.
2. Mark subagent claims as subagent findings, not user-confirmed facts.
3. If findings imply a decision change, propose a decision record.
4. Do not mark a new decision accepted until user confirmation.
5. Update `handoff.md` only if the next action, blocker, or current state changes.
