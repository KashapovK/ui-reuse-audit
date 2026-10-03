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

Audit material UI reuse and ownership before frontend implementation. Determine what the current consumer can actually reuse, what kind of missing capability remains, and where that capability belongs. Return that boundary to the downstream planning or implementation workflow; do not take over implementation.

The audit owns reuse and ownership classification. It does not own the detailed implementation plan, design-system mutation, Figma editing, Code Connect authoring, or unrelated architecture decisions.

## Activation boundary

Use this skill when all of the following are materially relevant:

- the task changes or introduces frontend UI in an existing codebase;
- a design system, UI kit, shared UI layer, adapter layer, reusable tokens/icons, analogous implementation, or Figma-to-code mapping may affect the implementation boundary;
- reuse, availability, or ownership is not already established well enough to implement without a material architectural guess.

Activation may be explicit (for example, "audit reuse before implementing this screen") or implicit in an implementation/planning request where the existing reusable system could materially change what should be built locally.

Do not activate for:

- copy-only edits, renames, formatting, or other mechanical changes;
- isolated visual adjustments whose owning component is already established and no reusable capability question is exposed;
- greenfield visual ideation with no existing reusable UI system to audit;
- non-UI work;
- implementation whose reuse and ownership decisions are already explicitly settled and no new material gap appears.

If a task begins as ordinary implementation but exposes an unresolved material reuse or ownership question, audit only that question and then return control to the original workflow.

## Keep the audit boundary narrow

The skill may:

- identify material UI capabilities that could change the implementation boundary;
- determine whether an existing reusable asset, adapter, composition pattern, or shared capability already covers the need;
- distinguish consumer-available capability from upstream-only capability;
- classify the ownership of missing UI capability;
- record version/availability gaps and missing external contracts separately from ownership;
- return a compact handoff for downstream planning.

The skill does not:

- implement or refactor UI;
- create local wrappers merely to make an implementation convenient;
- mutate a design system or publish a new package version;
- edit Figma or create/update Code Connect mappings;
- invent routes, APIs, mutations, actions, navigation targets, or other missing contracts;
- make unrelated product or architecture decisions;
- create, edit, or close tracker work unless separately authorized outside this audit.

Figma, GitHub, network access, and upstream repositories are optional evidence sources, not prerequisites for running the audit.

## Classify one primary ownership decision

For each material UI capability in scope, assign exactly one primary ownership decision. Availability and contract state are separate axes and must never be used as extra ownership decisions.

| Decision | Use when | Do not use when |
|---|---|---|
| `REUSE` | An existing reusable primitive, component, token, icon, shared pattern, or existing adapter already covers the need closely enough to use directly. | The required capability is not actually available to the consumer, or material translation/composition is still required. |
| `ADAPTER` | A reusable primitive exists, but a **new real integration boundary** is needed to translate framework/application concerns such as routing, image/runtime APIs, localization, theme/session context, or another environment-specific contract. | A suitable adapter already exists (that is `REUSE`), or the wrapper would only rename props, restyle, shorten imports, or hide a single call site. |
| `COMPOSE` | The requested feature/domain UI is an assembly of reusable primitives whose meaning, data flow, and layout belong to the feature or domain rather than to a generic shared primitive. | The missing capability itself is generic and reusable across consumers, or the result is specific to one application with no meaningful shared primitive boundary. |
| `EXTEND_DESIGN_SYSTEM` | A genuinely reusable, application-independent UI capability is missing and evidence supports shared design-system ownership across multiple consumers or repeated generic use. | The need is only visually similar to other screens, only hypothetical future reuse exists, or the capability contains domain/application behavior. |
| `APP_SPECIFIC` | The capability is intentionally tied to one application, product surface, or domain policy and should remain local even if it uses shared primitives internally. | The capability has a stable generic contract that belongs in the shared design system, or it is simply a feature-level composition of existing primitives. |

### Existing adapters are reuse

An adapter that already exists and satisfies the required integration is `REUSE`, not `ADAPTER`. `ADAPTER` means creating a new integration boundary because a reusable primitive cannot be consumed correctly without translating a real environment-specific concern.

Do not create an adapter to preserve a preferred local API, reduce import length, apply incidental styling, or wrap one consumer call site. Those are not sufficient ownership boundaries.

## Discover consumer reality before upstream possibility

Run discovery from the consuming application outward. The default evidence order is:

`consumer imports and call sites → manifest/workspace declarations → lockfile or resolved metadata → installed/workspace package exports and source → upstream design-system repository`

This order is a priority, not a requirement to inspect every layer. Stop as soon as the material claim is established strongly enough for the audit.

### 1. Start from the requested UI and current consumer usage

Identify only the material capabilities that could change what gets built or where it belongs. For each one, search the consumer first for:

- direct imports from the design-system/UI package;
- imports through local shared UI or adapter modules;
- existing call sites of likely reusable components, tokens, icons, or patterns;
- analogous feature implementations that may reveal an existing composition or application boundary.

Treat analogous feature code as evidence of current practice, not automatic proof of reusable ownership. A repeated application-local pattern can still be `COMPOSE` or `APP_SPECIFIC` rather than a design-system primitive.

Do not broaden the search into a full component inventory unless the task itself spans that surface.

### 2. Establish declared and resolved dependency state

When an external/workspace UI package may own the capability, identify the relevant consumer package boundary and record separately:

- the **declared constraint** from the manifest or workspace configuration;
- the **resolved version or workspace target** from the lockfile or authoritative package-manager metadata, when available;
- whether the dependency is external, workspace-linked, vendored, generated, or otherwise resolved through a nonstandard path.

Never call a semver range an exact installed version. If a lockfile or workspace link establishes the exact resolved state, prefer that for current-availability claims.

If several workspaces consume different versions or package surfaces, scope the audit to the actual consumer being changed instead of treating one manifest as repository-wide truth.

### 3. Verify the usable package surface

When availability is still material, inspect the resolved installed/workspace package or its authoritative exports/source to determine whether the consumer can actually use the capability.

Prefer evidence such as:

- package `exports` / entry points;
- generated or published type declarations;
- public barrel files;
- installed package source when that is the shipped surface;
- workspace package source at the exact linked revision/state.

A symbol appearing somewhere in repository source does not prove it is part of the public consumer surface. Conversely, absence from one barrel file does not prove absence if the package intentionally exposes secondary entry points.

Do not require network access when the local resolved package state already establishes the answer.

### 4. Inspect local reusable layers before declaring a gap

Before deciding that a shared capability is missing, check the relevant local reusable surfaces, proportionally:

- application adapters around the design-system primitive;
- shared UI wrappers that encode a real integration boundary;
- shared tokens/icons/assets;
- established feature-level compositions when the task appears domain-specific.

If an adapter already satisfies the integration, classify it as `REUSE`. Do not recommend a second wrapper merely because the upstream primitive is lower-level.

A local application component should not be promoted to design-system ownership solely because it is reused in more than one feature. Verify whether its contract is actually application-independent before considering `EXTEND_DESIGN_SYSTEM`.

### 5. Consult upstream only after consumer state is known

Use the upstream design-system repository, latest branch, package registry metadata, release notes, or another authoritative upstream source only when it answers a material question that consumer evidence cannot resolve, such as:

- whether a missing capability already exists upstream but is not in the consumer's resolved version;
- whether a consumer package is behind a release that introduced the needed public export;
- whether an apparent local gap is intentionally planned upstream.

Keep upstream state separate from consumer state. A component on upstream `main` or an unreleased branch is not directly reusable by a consumer that cannot import it.

When upstream evidence shows the capability exists beyond the consumer's current usable surface, classify availability as `UPSTREAM_ONLY` or `VERSION_GAP` as appropriate; do not silently convert the ownership decision into `REUSE` unless the consumer can actually use the capability now.

## Prove absence proportionally

A failed search is not proof that a capability is absent.

Before assigning `ABSENT`, inspect a representative set of the surfaces that would reasonably contain the capability, for example:

- likely package exports and secondary entry points;
- local shared UI/adapter directories;
- relevant tokens/icons/assets indexes;
- representative analogous implementations;
- the resolved package surface if an external UI package is involved.

Use the smallest representative search that makes the absence claim credible for this task. Do not scan unrelated packages or the entire repository merely to strengthen a low-risk claim.

If access limitations, package-manager state, generated artifacts, or ambiguous exports prevent a reliable conclusion, use `UNKNOWN` instead of `ABSENT`.

## Keep availability orthogonal to ownership

Ownership answers **where the capability belongs**. Availability answers **whether the current consumer can use the relevant capability now**. Do not collapse the two.

Use these availability statuses:

- `AVAILABLE` — evidence shows the current consumer can use the capability through a supported local or dependency surface now;
- `UPSTREAM_ONLY` — the capability exists in an upstream source but is not part of the consumer's current usable surface and no specific consumer-version delta is established;
- `VERSION_GAP` — the capability belongs to the dependency/design system and evidence establishes that a different released/resolved version or channel exposes it while the current consumer does not;
- `ABSENT` — representative evidence supports that the capability is not present in the relevant reusable surfaces;
- `UNKNOWN` — available evidence is insufficient to establish one of the above confidently.

A capability can therefore be, for example, `EXTEND_DESIGN_SYSTEM + ABSENT`, or `REUSE + AVAILABLE`. A component that exists only on design-system `main` is not `REUSE + AVAILABLE` for a consumer that cannot import it.

Use `VERSION_GAP` only when the version relationship itself is established. If the capability merely appears in an unreleased or otherwise unversioned upstream state, prefer `UPSTREAM_ONLY`.

## Keep evidence compact and claim-specific

For each material capability, retain only the evidence needed to support the ownership and availability classification. Useful evidence usually answers:

- where the consumer currently imports or would import from;
- what version/workspace state is actually resolved;
- whether the required symbol/API is exported and usable;
- whether an existing adapter/composition already owns the integration;
- what upstream evidence, if any, explains a gap.

Do not turn discovery into a repository inventory, dependency audit, or design-system catalog. The final output should be able to cite the decisive evidence without reproducing every search performed.

## Keep external contracts orthogonal to UI ownership

A route, API, mutation, action, navigation target, permission, or other non-UI dependency is not a sixth ownership decision.

Track external-contract state separately, for example:

- `AVAILABLE` — the required contract exists and is established for the task;
- `MISSING_CONTRACT` — the approved UI requires a contract that does not exist or is not yet provided;
- `UNKNOWN` — the evidence does not establish whether the contract exists.

Never invent a missing contract to make the UI appear complete. The UI ownership classification can still be established while the interaction remains blocked by `MISSING_CONTRACT` or `UNKNOWN`.

## Preserve the important distinctions

- **Consumer availability is not upstream existence.** Upstream source proves a capability exists somewhere, not that this consumer can use it.
- **Declared version is not resolved version.** A manifest range does not prove the exact installed package surface.
- **Exported capability is not repository-source existence.** A symbol present upstream or inside a package is not necessarily public to the consumer.
- **One search miss is not absence.** Use representative evidence or `UNKNOWN`.
- **Availability is not ownership.** A version gap does not decide whether a capability belongs to the design system or the application.
- **Missing backend/action state is not UI ownership.** Keep external contracts separate.
- **Primitive, adapter, composition, and application-specific UI are different boundaries.** Do not use wrappers or shared placement as substitutes for reasoning about those boundaries.
- **An audit result is not an implementation plan.** Classification should constrain downstream planning, not duplicate it.
- **Possible future reuse is not current shared ownership.** `EXTEND_DESIGN_SYSTEM` needs evidence of a generic reusable contract, not speculation.

## Use proportional depth

Inspect only the material capabilities whose classification could change what gets built or where it lives.

- Do not inventory an entire design system for a one-component task.
- Do not reopen capabilities whose reuse/ownership is already established unless new evidence creates a material conflict or gap.
- Stop investigating a candidate once the evidence is sufficient for its classification.
- Preserve `UNKNOWN` when missing evidence could change ownership or availability instead of filling the gap with a conventional default.
- If no material reuse/ownership question remains, stop the audit and return control to planning or implementation.

Consumer-first discovery is complete when each in-scope capability has enough evidence to support its current availability status and ownership classification, or an explicit `UNKNOWN` remains because the missing evidence could change the result.

## Compose with neighboring workflows

Use this audit as a pre-implementation boundary, then hand off rather than duplicating specialized workflows:

- **Figma design-to-code** may consume the audit result when implementing an approved design; this skill does not perform the implementation.
- **Code Connect workflows** may create or repair Figma-to-code mappings when separately requested; this skill may consume an existing mapping as evidence but does not author one.
- **Design-system/Figma library workflows** may implement a confirmed `EXTEND_DESIGN_SYSTEM` gap; this skill only establishes the ownership decision.
- **Decision-preflight or another architecture workflow** should handle a genuine unresolved technical/product choice that remains after factual reuse and ownership evidence is established.
- **Implementation planning** begins after the audit has established enough reuse, availability, and ownership constraints to avoid material guesses.

These neighboring workflows are optional. The audit must still produce a useful result when they are unavailable.

## Stop before implementation

The audit is complete when every in-scope material capability is either classified with sufficient evidence or explicitly left `UNKNOWN`, and any material version/availability or external-contract gaps are separated from ownership.

Do not continue into file-by-file planning, code changes, Figma mutation, package publication, or tracker lifecycle work. Return the established boundary to the workflow that owns those actions.
