---
name: ui-reuse-audit
description: Audit UI reuse and ownership before implementing frontend UI in an existing codebase. Use when a design system, UI kit, shared components, adapters, analogous implementations, or Figma mappings may materially affect what should be reused or where missing UI belongs. Verify what the current consumer can actually use before relying on upstream capability. Skip trivial copy/style changes, purely mechanical edits, greenfield visual ideation without an existing reusable system, and implementation work whose reuse/ownership is already settled.
license: MIT
metadata:
  author: KashapovK
  short-description: Audit UI reuse and ownership before implementation
  version: "0.1.0"
---

# UI Reuse Audit

Audit material UI reuse and ownership before frontend implementation. Establish what the current consumer already has access to, distinguish that from upstream-only capability, and return the smallest useful handoff for downstream planning.

This initial scaffold intentionally contains only the accepted scope boundary. The detailed taxonomy, discovery workflow, Figma evidence rules, missing-contract handling, output contract, and evaluation behavior are completed by the follow-up roadmap issues.

## Current boundary

- Inspect only material UI capabilities that could change implementation ownership.
- Prefer consumer reality over upstream possibility.
- Do not implement UI, mutate Figma, or create tracker work as part of the audit.
- Do not invent missing routes, APIs, actions, or navigation contracts.
- Stop when the reuse and ownership boundary is established well enough for downstream planning.
