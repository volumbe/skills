---
name: triage-bug
description: Investigate a reported bug, regression, or broken behavior by exploring the codebase, identifying the likely root cause, and producing a concrete fix and verification plan. Use when the user asks to triage a bug, debug an issue, find the cause of unexpected behavior, or understand what should be fixed before implementation.
---

# Triage a Bug

Use this skill to investigate a bug report and turn it into a high-signal diagnosis plus an implementation-ready repair plan.

This workflow should stay lightweight. Ask as little as needed up front, then investigate directly in the codebase and available runtime signals.

## Process

### 1. Capture the bug clearly

Start with the user's description of the bug.

If the report is vague, ask for the minimum missing detail needed to begin, such as:

- What is happening?
- What should happen instead?
- Where does it show up?
- Is there a repro, error message, failing test, or screenshot?

Do not stall on a long questionnaire. Start investigating as soon as you have a workable symptom.

### 2. Reproduce the problem or bound it tightly

Try to reproduce the bug when practical.

If full reproduction is expensive or blocked, find the nearest reliable signal instead:

- A failing test
- An error message or stack trace
- Logs or telemetry
- A broken UI flow
- A bad API response
- A data mismatch

Be explicit about what is confirmed versus what is still assumed.

### 3. Trace the relevant codepath

Explore the codebase from the symptom inward. Focus on:

- Entry points and request or event flow
- The modules most likely to own the broken behavior
- Existing tests and missing coverage around the failing path
- Configuration, schema, feature flag, and environment assumptions
- Recent changes to the affected area when history is useful
- Similar working code paths elsewhere in the repo

If the user explicitly asks for subagents or parallel work, delegate bounded investigation tasks with clear ownership. Otherwise, investigate locally.

### 4. Find the likely root cause

Work past the surface symptom and identify why the system behaves this way.

Classify the failure when helpful, for example:

- Regression from a recent change
- Missing guardrail or validation
- Incorrect default or fallback
- State synchronization bug
- Race condition or ordering issue
- Data contract mismatch
- Cache invalidation or stale state problem
- Environment or configuration drift

Prefer the smallest durable fix over a symptom patch.

### 5. Produce a triage report

Return a concise write-up using this structure:

## Problem Summary

- Actual behavior
- Expected behavior
- Reproduction steps or strongest available evidence

## Root Cause Analysis

- The code path or subsystem involved
- Why the bug happens
- Any contributing factors or adjacent risks

## Fix Plan

- The minimal repair that should solve the problem
- The files, modules, or surfaces likely to change
- Tradeoffs, follow-up risks, or migration concerns

## Validation Plan

- The checks needed to prove the fix works
- Regression checks for nearby behavior

## Open Questions

- Unknowns that still block certainty
- The next best discriminating check if root cause is not fully proven

If you cannot prove a single root cause, give the top hypotheses in priority order with the evidence for each one.

### 6. Keep the default outcome simple

The default outcome of this skill is a concise diagnosis, likely root cause, and a concrete repair and validation plan. Only create tickets or use a test-first workflow when the user explicitly asks for that.

## Good outcomes

- The user gets a likely root cause, not just a symptom summary
- The recommended fix is implementation-ready
- Adjacent risk is called out clearly
- Uncertainty is explicit when the evidence is incomplete

## Example prompts this skill should handle well

- "Triage why this settings form sometimes saves stale data"
- "Help me figure out why this API route started returning 500s"
- "Investigate this regression before we start fixing it"
- "Find the likely cause of this flaky checkout bug"
