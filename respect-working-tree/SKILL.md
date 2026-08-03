---
name: respect-working-tree
description: |
  Prevents a coding agent (Claude Code, GitHub Copilot, or any Agent Skills-compatible
  agent) from reverting or re-applying changes the user has already made manually to
  their working tree. Use when the agent begins a response by verifying or "re-applying"
  edits that the user intentionally made themselves — the agent should trust the working
  tree as the source of truth, not its own session history. Also covers the pattern of
  always reading a file before editing to avoid fighting over state.
author: Skiller
version: 1.1.0
date: 2026-08-03
---

# Respect the Working Tree

## Problem

Coding agents keep a mental model of what files look like from their own edits during a
session. When the user manually edits a file outside of the agent's view (e.g., in the
editor, via git operations, or by reverting a change), the agent may try to "re-apply"
its version of the change — effectively overwriting what the user intentionally did.

This creates a tug-of-war over the working tree and is frustrating for developers who want
to collaborate on the code, not just accept AI output wholesale.

## Context / Trigger Conditions

- User says "STOP", "don't change that", "I made that change myself", or "leave it"
- User manually edits a file that the agent previously modified
- User asks "what's next?" and the agent starts re-verifying previous changes instead of moving forward
- File content on disk doesn't match what the agent wrote in the last session
  (this is normal — sessions are stateless; the file on disk is always correct)
- The agent notices a file looks "old" compared to its session memory

## Solution

1. **The working tree is always the source of truth.** If a file doesn't match what
   the agent wrote last session, assume the user changed it intentionally.

2. **Never re-apply a change without being asked.** If the agent thinks a change is
   missing, it should ask: *"I notice X doesn't have Y — did you intentionally revert
   that?"* — not silently re-apply it.

3. **Always read before editing.** Read the current file state with your agent's
   file-reading tool (`Read` in Claude Code, `get_file` in Copilot) before making any
   change. Never assume the file matches a prior session's output.

4. **Move forward, not backward.** When a user asks "what's next?", answer that question.
   Don't use it as a trigger to verify or re-do prior work.

5. **Session memory ≠ file state.** The agent's memory of "what I wrote" is irrelevant once
   the session ends. The file on disk is what matters.

## Verification

Ask yourself before any file edit:
- Did I just read this file from disk?
- Is the user asking me to change this, or am I doing it because I *think* it should match my prior work?
- If the user's version differs from mine, did I ask them about it?

## Example

**Bad pattern:**
> User: "What's next?"
> Agent: "Let me verify the #2566 changes are actually in the file before moving on..."
> *[Re-reads file, sees user's version, re-applies its own version anyway]*

**Good pattern:**
> User: "What's next?"
> Agent: "Next up is #2564 — Surveyor Name autocomplete. Here's what it needs..."

**Bad pattern:**
> User: "STOP"
> Agent: *[Continues applying changes anyway on the next turn]*

**Good pattern:**
> User: "STOP"
> Agent: "Stopped. Waiting for your instructions."

## Notes

- This is especially important in long sessions where multiple back-and-forth edits have
  occurred. The user's last manual edit is intentional, not an error to correct.
- The user may revert agent changes deliberately (to test, to simplify, or because
  they preferred a different approach) — that is their right.
- Git commits are also the user's domain. Never stage or commit without explicit instruction.
  If changes are uncommitted, that is also intentional — do not push or commit them.

## References

- Copilot instructions: `.github/copilot-instructions.md`
- Claude Code instructions: `CLAUDE.md`

## Activation History
- 2026-02-28: Session where Copilot re-applied PwDatePicker changes the user had already made manually, and re-verified #2567 changes the user had committed themselves.
