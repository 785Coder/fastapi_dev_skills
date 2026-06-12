---
name: backend-dev-rules
description: Backend project development rules for implementing, modifying, reviewing, or debugging backend services and APIs. Use when Codex works in a backend codebase and must follow project conventions for comments, minimal changes, communication, scope control, reset decisions, pragmatic tradeoffs, and uv-based project commands.
---

# Backend Dev Rules

Use this skill as the project-level operating contract for backend work. Prefer the user's latest instruction over earlier assumptions, and apply these rules in proportion to the risk and urgency of the task.

## Comments And Docs

- Explain why code exists, not what each line does. Use comments for business reasoning, design decisions, compatibility constraints, and technical tradeoffs.
- Keep comments short and necessary. Do not annotate self-evident operations.
- For core services and API interfaces, make input and output structures clear when the surrounding code does not already do so. Include field names, types, and meaning where it helps future maintainers, for example `{user_id: int, username: str, email: str}`.

## Minimal Change Scope

- Modify only files and lines directly related to the current task.
- Respect working legacy code. Do not opportunistically refactor unrelated code just because it could be cleaner.
- Avoid whole-file formatting or broad style churn when making a small behavior change.
- Keep version-control diffs reviewable. Separate necessary behavior changes from incidental cleanup.

## Communication And Consent

- Ask before acting when the requirement is ambiguous, key context is missing, or several equally valid technical paths exist.
- Prefer decision-oriented questions. Offer concrete options and tradeoffs, such as a simpler implementation versus a higher-performance implementation.
- Confirm before high-risk operations, including large refactors, deleting core code, overwriting important files, or changing public API behavior.
- Do not add unrequested features, behavior changes, or interface adjustments silently. State the likely impact and get user confirmation first.

## Resetting Direction

- Treat the user's newest instruction as authoritative, even when it supersedes earlier discussion.
- If the user proposes a new approach or the current attempt is stuck, discard the obsolete approach instead of preserving it for consistency with earlier turns.
- Do not force old code, failed attempts, or previous conversation decisions into the final implementation when a clean restart better serves the current goal.

## Pragmatic Flexibility

- Treat these rules as development guidance, not rigid blockers.
- If the user explicitly asks to ignore standards temporarily, quickly get something running, use hard-coded data, or produce a rough draft, prioritize that immediate goal.
- Match the user's pace. Provide incremental changes for step-by-step work, or a complete solution when the user asks for one.

## Project Commands

Use `uv` for project commands:

- Run the project with `uv run --no-sync python main.py`. Keep `--no-sync` to avoid overwriting a customized `tortoise-orm`.
- Install dependencies with `uv sync`.
- Add a package with `uv add <package-name>`.
- Install the project in editable mode with `uv pip install -e .`.
