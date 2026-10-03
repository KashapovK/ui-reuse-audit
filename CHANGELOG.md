# Changelog

All notable changes to this project will be documented in this file.

## 0.1.0 - 2026-10-03

### Added

- Initial skills-only `ui-reuse-audit` plugin and Agent Skill package.
- Activation boundary for material frontend UI reuse/ownership questions, including negative triggers for trivial and already-settled work.
- Five primary ownership decisions: `REUSE`, `ADAPTER`, `COMPOSE`, `EXTEND_DESIGN_SYSTEM`, and `APP_SPECIFIC`.
- Consumer-first discovery workflow covering imports/call sites, manifests/workspaces, resolved dependency state, installed/workspace exports, local adapters/shared UI, and upstream evidence.
- Availability states `AVAILABLE`, `UPSTREAM_ONLY`, `VERSION_GAP`, `ABSENT`, and `UNKNOWN`, kept separate from ownership.
- Optional Figma/library/Code Connect evidence handling without Figma mutation or mapping creation.
- Missing external-contract handling with `MISSING_CONTRACT`/`UNKNOWN`, no invented route/API/action contracts, and honest unavailable UI states.
- Compact audit output contract with traceable evidence, implementation-neutral required actions, and explicit stop conditions before implementation.
- Markdown activation and behavioral evaluation cases covering all primary decisions, version gaps, upstream-only capability, missing contracts, insufficient evidence, and proportional stopping.
- Public installation, usage, composition, development, and security documentation.
- CI validation for Agent Skills structure, plugin manifest consistency, release metadata, whitespace, and secret scanning.
