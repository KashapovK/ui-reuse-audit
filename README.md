# UI Reuse Audit

> **Reuse before reimplementation.**

**UI Reuse Audit is a consumer-first evidence gate for frontend UI work.** It checks what reusable UI already owns a requirement, what the current consumer can actually use, where any remaining gap belongs, and which external contracts must not be invented before implementation starts.

It is an instruction-only Agent Skill. It does not require an MCP server, credentials, Figma, GitHub, or network access when local repository evidence is sufficient.

## The problem

Frontend implementation can easily duplicate an existing design-system capability when the reusable boundary is not checked first:

- a screen reimplements a component that already exists in the UI kit;
- an existing framework adapter is bypassed and replaced with another wrapper;
- a feature composition is promoted into the design system without a generic contract;
- a component found on upstream `main` is treated as importable even though the current consumer resolves an older package;
- a Figma component or Code Connect mapping is mistaken for proof that the consumer can import the matching code;
- a missing route, API, mutation, action, or navigation target is guessed so the UI can appear complete.

`ui-reuse-audit` runs before local component design. It establishes reusable ownership, consumer availability, and any blocking contract gaps, then hands that boundary to the workflow that will plan or implement the UI.

## Quick start

Install the skill in the current project:

```bash
npx skills add KashapovK/ui-reuse-audit --skill ui-reuse-audit
```

Install it for the current user across projects:

```bash
npx skills add -g KashapovK/ui-reuse-audit --skill ui-reuse-audit
```

Try it without installing:

```bash
npx skills use KashapovK/ui-reuse-audit --skill ui-reuse-audit
```

After a stable release tag is published, you can pin the install to that release, for example:

```bash
npx skills add https://github.com/KashapovK/ui-reuse-audit/tree/v0.1.0 --skill ui-reuse-audit
```

The repository also contains `.codex-plugin/plugin.json`, which packages the same skill as a skills-only Codex compatibility plugin. It declares only the `skills/` directory and read capability; there are no bundled MCP servers, apps, or hooks. When distributed through the Plugins directory, install the plugin from that directory. For local plugin development/testing, register the plugin root through a supported local or personal plugin source.

## When to use it

Use the skill before implementing or planning frontend UI when an existing codebase may already provide a material part of the requested interface through:

- a design system or UI kit;
- shared components, tokens, icons, or assets;
- framework/application adapters;
- analogous feature implementations;
- Figma library identity or existing Code Connect mappings;
- a versioned package whose actual consumer availability is unclear.

The skill may activate explicitly or implicitly. A typical implicit case is a Figma-to-code task in an application that already consumes a design-system package.

It can also re-enter a settled implementation for one newly exposed gap, such as discovering that the planned shared component is not exported by the package version the application actually resolves.

## When it stays out

Do not use it for:

- copy-only edits, renames, formatting, or other mechanical changes;
- isolated CSS adjustments when the owning component is already established;
- non-UI work;
- greenfield visual ideation with no existing reusable UI system;
- implementation whose reuse and ownership decisions are already settled and no new material gap appears.

The skill is not a general frontend architecture review and is not a replacement for ordinary implementation planning.

## The five ownership decisions

Each material UI capability gets one primary decision. Availability and external-contract state are separate axes.

| Decision | Meaning |
|---|---|
| `REUSE` | An existing primitive, component, token, icon, shared pattern, or existing adapter already owns the capability. A version/release gap may still prevent immediate use. |
| `ADAPTER` | A reusable primitive exists, but a new real framework/application integration boundary is needed, such as routing, localization, runtime image APIs, theme/session context, or another environment-specific contract. |
| `COMPOSE` | The requirement is a feature/domain assembly of existing reusable pieces; its data, wording, orchestration, policy, or layout belongs to the feature/domain. |
| `EXTEND_DESIGN_SYSTEM` | A genuinely missing, application-independent reusable primitive belongs in the shared design system. |
| `APP_SPECIFIC` | The capability intentionally belongs to one application/domain and should remain local, while still reusing shared primitives where possible. |

An existing adapter is always `REUSE`, not `ADAPTER`. A new wrapper that only renames props, shortens imports, applies incidental styling, or hides one call site is not a legitimate adapter boundary.

## Consumer-first evidence

The default evidence priority is:

```text
consumer imports/call sites
→ manifest/workspace declarations
→ lockfile or resolved dependency state
→ installed/workspace exports and source
→ upstream design-system repository
```

The audit is demand-driven. It investigates only material capabilities from the requested UI rather than inventorying the entire repository or design system.

Important distinctions:

- a manifest range is not an exact installed version;
- repository source is not necessarily a public package export;
- one failed search is not proof of absence;
- analogous application code is precedent, not automatic shared ownership;
- upstream existence is not current consumer availability.

When local imports, lockfiles, installed packages, workspaces, adapters, and exports are enough to establish the answer, the audit works without GitHub or network access.

## Availability and version gaps

Availability is reported separately from ownership:

| Availability | Meaning |
|---|---|
| `AVAILABLE` | The current consumer can use the capability through a supported local or dependency surface now. |
| `UPSTREAM_ONLY` | The capability exists upstream but is not part of the current consumer surface, and no specific released-version delta is established. |
| `VERSION_GAP` | Evidence establishes that another released/resolved version or channel exposes the reusable capability while the current consumer does not. |
| `ABSENT` | Representative evidence supports that the capability is not present in the relevant reusable surfaces. |
| `UNKNOWN` | Available evidence is insufficient to establish one of the above confidently. |

A version gap does not create a new component ownership decision. For example:

- current package exports `Button` → `REUSE + AVAILABLE`;
- current package lacks `Dialog`, but a newer published design-system release exports it → `REUSE + VERSION_GAP`;
- `Dialog` exists only on unreleased design-system `main` → `REUSE + UPSTREAM_ONLY`;
- no generic primitive exists after representative checks and the contract is application-independent → `EXTEND_DESIGN_SYSTEM + ABSENT`.

This prevents a consumer from locally recreating a capability that already belongs to the design system merely because its dependency is behind.

## Figma and Code Connect

Figma is optional evidence, not a prerequisite.

The audit may use existing:

- Figma component or component-set identity;
- library provenance;
- variables/styles that map to shared tokens;
- Code Connect mappings;
- documented Figma ↔ code relationships.

That evidence can establish design intent or a likely code counterpart, but it does not prove current consumer availability. The resolved package/export surface still decides whether the application can use the capability now.

The skill does not generate code from Figma, create or repair Code Connect mappings, mutate Figma libraries, or reconcile an entire Figma design system. Those actions belong to specialized workflows.

## Missing routes, APIs, actions, and other contracts

External contracts are tracked separately from UI ownership:

- `AVAILABLE` — the required contract exists and is established;
- `MISSING_CONTRACT` — the approved UI depends on a contract that does not exist, is deferred, or has not been provided;
- `UNKNOWN` — available evidence does not establish the approved contract.

For `MISSING_CONTRACT`, the skill does **not** invent an endpoint, route, mutation name, payload, return type, navigation target, permission rule, service API, or fake success behavior.

Instead it keeps the approved UI as complete as the known contract allows, records a concrete TODO for the missing dependency, and uses a disabled or otherwise honestly unavailable state when an active control would falsely imply working behavior. A different approved unavailable state may be used instead of a disabled control.

## What it returns

The default handoff is one compact table:

```text
| UI element | Evidence | Availability | Decision | Required action | Ownership |
```

One row represents one material capability, not one DOM node, Figma layer, source file, or visual detail.

Representative result:

| UI element | Evidence | Availability | Decision | Required action | Ownership |
|---|---|---|---|---|---|
| Primary button | current package export and existing call sites | `AVAILABLE` | `REUSE` | reuse | shared UI package |
| Confirmation dialog | newer published package release exports the matching generic dialog; current resolved version does not | `VERSION_GAP` | `REUSE` | align/upgrade dependency | shared design system |
| Settings form assembly | shared inputs/buttons exist; wording and data flow are feature-owned | `AVAILABLE` | `COMPOSE` | compose locally | feature/domain layer |

Optional sections appear only when materially needed:

### Missing contracts

- **Save settings** — `MISSING_CONTRACT`: no approved mutation/action exists. Keep the approved visual state, leave a concrete TODO for the missing save contract, and do not expose a fake working action.

### Verification notes

Use this section only when missing evidence could change `Availability` or `Decision`. The audit keeps the affected field `UNKNOWN` rather than guessing and names the smallest decisive verification step.

## How it composes with other workflows

`ui-reuse-audit` is a pre-implementation boundary, not an orchestrator for every adjacent tool.

- **Figma design-to-code** implements an approved design after reuse/ownership is known.
- **Code Connect** workflows create or repair mappings when separately requested; this audit only consumes existing mappings as evidence.
- **Design-system/Figma reconciliation** workflows implement or reconcile a confirmed `EXTEND_DESIGN_SYSTEM` gap.
- **Decision/architecture workflows** handle genuine unresolved technical or product choices that remain after factual reuse evidence is established.
- **Implementation planning** consumes the compact audit handoff and decides concrete files, APIs, component structure, and sequencing.

No companion skill is required. Figma, GitHub, and external research remain optional evidence sources.

## Example prompts

Explicit audit:

```text
Audit this UI task for reuse and ownership before implementation. Check the current consumer package, local adapters, and analogous feature code before proposing new components.
```

Figma-driven task:

```text
Before implementing this Figma screen, verify which material pieces already map to our design system or adapters and whether the current application version can actually use them.
```

Version-sensitive task:

```text
This component exists on the design-system repository, but I do not know whether our resolved package exports it. Audit the consumer state before we create anything local.
```

Missing-contract task:

```text
Audit the approved form UI. If the save route/API/action is not established, do not invent it; separate the missing contract from the UI ownership result.
```

## Project instructions

A consuming repository can add a short routing rule to `AGENTS.md` without copying the whole procedure:

> Use `ui-reuse-audit` before planning or implementing material frontend UI when an existing design system, UI kit, shared component, adapter, analogous implementation, or Figma mapping could change what should be built or where it belongs. Verify the current consumer surface before relying on upstream capability, keep external contracts separate, and skip trivial/mechanical changes or already-settled ownership.

## Plugin/package model

The repository is intentionally skills-only for `v0.1.0`:

```text
.codex-plugin/plugin.json
skills/ui-reuse-audit/SKILL.md
tests/
```

The compatibility manifest declares `skills: "./skills/"` and only read capability. There are no MCP servers, apps, hooks, runtime services, or required credentials.

The skill itself follows the Agent Skills `SKILL.md` format, so it can be installed independently through a compatible skills CLI or consumed as part of the plugin package.

## Privacy and security

Prefer local consumer evidence first. Do not send private source code, credentials, internal endpoints, personal data, proprietary Figma content, or other sensitive material to external search/documentation services merely to strengthen an audit.

When a connected repository, Figma source, or other private system is used, preserve its authorization boundary and retrieve only the evidence needed for the classification. See [SECURITY.md](SECURITY.md).

## Development

Validate the skill with the repository's pinned Agent Skills reference validator:

```bash
skills-ref validate ./skills/ui-reuse-audit
```

The CI workflow also validates the compatibility plugin manifest, checks release-version consistency and whitespace, and scans for secrets. These are static checks; they are not behavioral evidence.

Behavioral coverage is documented in:

- [activation cases](tests/activation-cases.md);
- [behavioral cases](tests/behavior-cases.md).

Real-project validation is tracked separately from the static and markdown evaluation definitions.

## Project information

- [Agent Skills specification](https://agentskills.io/specification)
- [OpenAI plugin packaging](https://developers.openai.com/plugins/build/plugins)
- [MIT license](LICENSE)
- [Security policy](SECURITY.md)
- [Changelog](CHANGELOG.md)
