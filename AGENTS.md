# AGENTS.md — employee-internal-transfer

Vendor-agnostic governance for this repository. This file, together with `.agents/skills/`, makes the repository self-contained and independent of any specific AI tool or provider (Claude, Gemini, Cursor, Windsurf, Copilot, etc.).

## INT AI-First Engineering Policy

This project is developed under the INT AI-First Spec-Driven Development (SDD) methodology. All feature work is derived from an approved Business Requirements Document (BRD), formalized as a spec, peer-reviewed, planned, broken into tasks, built test-first, and code-reviewed before release. No implementation work proceeds without a corresponding approved spec.

## Authority & Resolution Hierarchy

Whenever any skill or governance rule is invoked in this repository, resolution MUST follow this priority order:

1. **Priority 1 — Local Repository First**: Check `AGENTS.md` (this file) and local project skills at `.agents/skills/<skill_name>/SKILL.md`. If present, load and execute the local project skill/rule first.
2. **Priority 2 — Global Fallback Second**: Only if a requested skill or rule is not present locally, fall back to the global skill location matching whichever AI tool is in use:
   - Claude Code: `~/.claude/skills/<skill_name>/SKILL.md`
   - Gemini: `~/.gemini/config/skills/<skill_name>/SKILL.md`

This project is intentionally vendor-agnostic (per the top of this file) and supports both fallback locations so it works the same way regardless of which AI tool a contributor uses. The org-wide `.agent/` Control Plane files copied into this repo are authoritative and were left byte-identical to their source, which references only the Gemini path (e.g. in `.agent/rules/int-standards.md`) — the dual-fallback rule above is this repo's own governance and takes precedence for contributors using Claude Code.

Organization-wide INT SDD rules referenced from `.agent/` govern engineering process but MUST NOT override project-specific constraints recorded in `.ai-context/constitution.md` once a BRD has been ingested.

## SDD Lifecycle Definition

```
BRD → Gate 0 (BRD Review) → Spec (.spec.md) → Gate 1 (Spec Peer Review)
    → Plan (.plan.md) → Tasks (.tasks.md) → Test Cases (.test_cases.md)
    → TDD RED (tests/) → TDD GREEN (src/) → Test Verification
    → Gate 2 (Code Review) → Release (RELEASE-vX.Y.Z.md)
```

- **Gate 0**: `.ai-context/BRD.md` must be reviewed and marked `Approved` before any spec drafting begins.
- **Gate 1**: Spec Peer Review evaluating requirement completeness, technical approach, acceptance criteria, and development readiness.
- **Gate 2**: Code Review evaluating implementation against the approved spec, code quality, security, and test coverage.
- Live status of every spec is tracked in `.ai-context/status.md`.
- A spec `Rejected` or marked `Changes Requested` at Gate 1 blocks all planning, tasking, test-case drafting, and implementation until it is revised and re-approved.
- Formal Change Request workflow (spec revision + Gate 1 re-approval) is triggered only when a prompt explicitly contains "Change Request" or "CR"; other prompts are treated as development fixes under the active approved spec.

## Core Governance Rules

- Never hardcode secrets, API keys, or credentials — read from environment variables.
- Validate and sanitize all incoming request payloads before processing.
- Wrap external calls and I/O in proper error handling; never swallow errors silently; log via the project's standard logger.
- Never include comments or commit messages indicating code was AI-generated.
- Do not add AI-tool-specific configuration files (e.g. `.cursorrules`, `.copilotignore`) outside `.agent/`.
- Authenticated Git user email (`git config user.email`) must match the assigned reviewer roster below before that reviewer's PR approval actions are accepted; name matching is not evaluated.
- Pulling/reading code does not grant approval rights — role separation between developer and reviewer is enforced.
- Every completed task, code generation, or significant analysis must be appended (never overwritten) to `.ai-context/prompt_history.md`.

## Reviewer Roster

| Gate | Reviewer(s) |
|---|---|
| Gate 0 (BRD Review) | supratim.jetty@intglobal.com |
| Gate 1 (Spec Peer Review) | soumyadeep.adhikary@intglobal.com |
| Gate 2 (Code Review) | TBD — assign before first Gate 2 review |

## Related Skills

- Project setup: `.agents/skills/int-project-setup/SKILL.md`
- Feature lifecycle: `.agents/skills/int-sdd-lifecycle/SKILL.md`
- BRD ingestion: `.agents/skills/int-brd-ingestion/SKILL.md`
- Incident management: `.agents/skills/int-incident-management/SKILL.md`
- Hotfix management: `.agents/skills/int-hotfix-management/SKILL.md`
- Release management: `.agents/skills/int-release-management/SKILL.md`
- Session continuation: `.agents/skills/int-session-continuation/SKILL.md`
