# UI Reuse Audit

`ui-reuse-audit` is an instruction-only Agent Skill for checking UI reuse, consumer availability, and ownership before frontend implementation in an existing codebase.

The skill is being developed incrementally under the repository roadmap. The initial `0.1.0` scaffold establishes packaging, metadata, validation, and the accepted scope boundary; the detailed audit workflow is completed by the follow-up issues.

## Intended use

Use the skill when a frontend task may already be covered by an existing design system, UI kit, shared component, adapter, analogous implementation, or Figma mapping and that fact could change where or how the UI should be implemented.

The audit is consumer-first: capability available to the current application is not the same thing as capability that merely exists in an upstream repository.

## Non-goals

The skill does not implement UI, mutate Figma, create Code Connect mappings, perform a broad design-system rebuild, or replace general architecture/planning workflows.

It also does not invent missing routes, APIs, actions, or navigation contracts in order to make an approved control appear functional.

## Repository structure

```text
.codex-plugin/plugin.json
.github/workflows/validate.yml
skills/ui-reuse-audit/SKILL.md
tests/
README.md
CHANGELOG.md
LICENSE
SECURITY.md
```

## Status

Development is tracked in GitHub issues starting from roadmap issue #1.

Current target: `v0.1.0`.

## License

MIT
