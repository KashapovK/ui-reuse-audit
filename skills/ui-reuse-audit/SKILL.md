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
- consume existing Figma/library/Code Connect evidence when it helps establish identity or ownership;
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

## Use Figma as optional ownership evidence

When the task is driven by a Figma design, consume Figma evidence only to answer material reuse/ownership questions. Do not turn the audit into a Figma implementation or reconciliation workflow.

Useful evidence can include:

- component or component-set identity from a published Figma library;
- variable/style identity that maps to reusable tokens;
- existing Code Connect mappings;
- existing documented Figma ↔ code relationships;
- library provenance showing that the design asset belongs to the same design-system family as the consumer package.

Treat this evidence according to what it actually proves:

- a Figma library component can establish **design intent** and a likely reusable owner;
- a Code Connect mapping can strongly support the intended code counterpart;
- a variable/style mapping can support reuse of a token or shared visual primitive;
- none of these alone proves that the current consumer's resolved package exports the required code capability.

Consumer availability must still be verified through the consumer-first workflow. If Figma maps to a code component that exists only upstream, the result remains `UPSTREAM_ONLY` or `VERSION_GAP` rather than `AVAILABLE`.

Do not infer ownership from visual similarity alone. Two visually similar objects can have different domain contracts; conversely, a Figma instance with a different local label can still map to an existing reusable primitive.

If Figma metadata is unavailable, stale, ambiguous, or detached, continue with code/package evidence and record the Figma side as unverified only when that uncertainty materially affects the conclusion.

### Delegate specialized Figma work

This skill may consume existing Figma evidence, but it must not:

- inspect or reproduce the full internal workflow of a Figma design-to-code implementation skill;
- create or repair Code Connect mappings;
- mutate Figma components, variables, styles, or libraries;
- reconcile an entire Figma library against a code design system;
- generate implementation code from Figma.

When the audit identifies a need for one of those actions, record the ownership/reuse conclusion and hand off to the specialized workflow that owns it.

## Resolve ownership gaps without wrapper proliferation

When consumer discovery shows that no directly reusable surface fully covers the requirement, classify the remaining gap by its actual contract, not by convenience or folder structure.

### Choose `ADAPTER` only for a real integration boundary

Use `ADAPTER` when all of the following are true:

- an existing reusable primitive owns the generic UI behavior;
- the consumer cannot use it correctly without translating an environment/framework/application integration concern;
- that translation has a stable boundary that can serve more than a single incidental call site.

Typical evidence includes framework routing primitives, runtime image APIs, localization objects, theme/session context, platform-specific accessibility hooks, or other environment contracts that the shared primitive should not import directly.

If an existing adapter already performs the translation, use `REUSE`. If the proposed wrapper only renames props, fixes styling, re-exports a symbol, shortens an import, or hides one call site, do not classify it as `ADAPTER`.

### Choose `COMPOSE` for feature/domain assemblies

Use `COMPOSE` when the UI requirement is a meaningful feature/domain assembly of existing reusable pieces and the assembly's data, wording, policy, orchestration, or layout belongs to that feature/domain.

A composition can be reused across several screens inside the application without becoming a design-system primitive. Local repetition is evidence to inspect, not automatic evidence for `EXTEND_DESIGN_SYSTEM`.

### Choose `EXTEND_DESIGN_SYSTEM` only for a reusable primitive gap

Use `EXTEND_DESIGN_SYSTEM` only when the missing capability has a stable application-independent contract and evidence supports shared ownership, for example:

- the same generic interaction/visual primitive is needed across unrelated application features or multiple consumers;
- Figma library/design-system evidence shows it is intended as a reusable library asset rather than a one-screen composition;
- existing design-system primitives establish a consistent ownership boundary that the new capability naturally extends;
- the required behavior can be specified without importing application routes, domain services, business state, or product-specific policy.

Do not use `EXTEND_DESIGN_SYSTEM` merely because the element appears in Figma, is visually polished, could hypothetically be reused later, or would make local code shorter.

For composite widgets such as dialogs, keep generic mechanics in the design-system ownership only when the contract is truly generic. Application orchestration, service calls, routing, analytics policy, domain state, and business-specific content remain outside that primitive.

### Choose `APP_SPECIFIC` for intentional local capability

Use `APP_SPECIFIC` when the capability is coupled to application/domain behavior strongly enough that extracting it would either leak product concepts into the design system or require a generic API invented only to hide local behavior.

It can still reuse tokens, primitives, and adapters. `APP_SPECIFIC` does not mean "build everything from scratch"; it means the owning composition/behavior remains local.

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
- whether Figma/Code Connect identifies an intended reusable counterpart;
- what upstream evidence, if any, explains a gap.

Do not turn discovery into a repository inventory, dependency audit, design-system catalog, or Figma reconciliation report. The final output should be able to cite the decisive evidence without reproducing every search performed.

## Handle missing external contracts without inventing them

A route, API, mutation, action, navigation target, permission, service method, or other non-UI dependency is not a sixth ownership decision.

Track external-contract state separately:

- `AVAILABLE` — the required contract exists and is established for the task;
- `MISSING_CONTRACT` — the approved UI requires a contract that does not exist, is explicitly deferred, or has not been provided;
- `UNKNOWN` — available evidence does not establish whether the contract exists or what its approved shape is.

When a required contract is `MISSING_CONTRACT`:

1. keep the UI ownership classification independent from that gap;
2. do not invent an endpoint, route, mutation name, payload, return type, navigation target, service API, permission rule, or fake success behavior;
3. preserve the approved visual state as far as the established UI contract allows;
4. record a concrete TODO that names the missing dependency and where downstream implementation must reconnect it once the contract exists;
5. use a disabled control or another explicitly non-interactive approved state when presenting an active control would falsely imply working behavior;
6. do not replace the missing action with unrelated local state, placeholder navigation, console logging, a no-op callback, or a guessed temporary contract unless the user/spec explicitly requires such a prototype behavior.

A disabled control is not mandatory when the approved design already defines another honest unavailable state, or when the control itself should be omitted until the contract exists. The key requirement is that the UI must not represent a false interaction.

When the contract is `UNKNOWN`, avoid guessing. If ownership can still be decided, finish the audit with the uncertainty recorded. If the unknown contract materially changes UI ownership, leave the affected capability `UNKNOWN` and hand the unresolved decision back to the appropriate planning/product workflow.

The missing-contract finding must fit alongside the same UI capability evidence; it must not cause a second implementation plan or a speculative backend design.

## Preserve the important distinctions

- **Consumer availability is not upstream existence.** Upstream source proves a capability exists somewhere, not that this consumer can use it.
- **Declared version is not resolved version.** A manifest range does not prove the exact installed package surface.
- **Exported capability is not repository-source existence.** A symbol present upstream or inside a package is not necessarily public to the consumer.
- **Figma identity is not consumer availability.** A library or Code Connect match identifies intent/counterpart, not necessarily an importable current package surface.
- **Visual similarity is not ownership.** Reuse/extension decisions require contract evidence, not appearance alone.
- **One search miss is not absence.** Use representative evidence or `UNKNOWN`.
- **Availability is not ownership.** A version gap does not decide whether a capability belongs to the design system or the application.
- **Missing backend/action state is not UI ownership.** Keep external contracts separate.
- **Primitive, adapter, composition, and application-specific UI are different boundaries.** Do not use wrappers or shared placement as substitutes for reasoning about those boundaries.
- **An audit result is not an implementation plan.** Classification should constrain downstream planning, not duplicate it.
- **Possible future reuse is not current shared ownership.** `EXTEND_DESIGN_SYSTEM` needs evidence of a generic reusable contract, not speculation.

## Use proportional depth

Inspect only the material capabilities whose classification could change what gets built or where it lives.

- Do not inventory an entire design system for a one-component task.
- Do not perform broad Figma library reconciliation when one component identity is enough.
- Do not reopen capabilities whose reuse/ownership is already established unless new evidence creates a material conflict or gap.
- Stop investigating a candidate once the evidence is sufficient for its classification.
- Preserve `UNKNOWN` when missing evidence could change ownership or availability instead of filling the gap with a conventional default.
- If no material reuse/ownership question remains, stop the audit and return control to planning or implementation.

Consumer-first discovery is complete when each in-scope capability has enough evidence to support its current availability status and ownership classification, or an explicit `UNKNOWN` remains because the missing evidence could change the result.

## Compose with neighboring workflows

Use this audit as a pre-implementation boundary, then hand off rather than duplicating specialized workflows:

- **Figma design-to-code** may consume the audit result when implementing an approved design; this skill does not perform the implementation.
- **Code Connect workflows** may create or repair Figma-to-code mappings when separately requested; this skill may consume an existing mapping as evidence but does not author one.
- **Design-system/Figma library workflows** may implement or reconcile a confirmed `EXTEND_DESIGN_SYSTEM` gap; this skill only establishes the ownership decision.
- **Decision-preflight or another architecture workflow** should handle a genuine unresolved technical/product choice that remains after factual reuse and ownership evidence is established.
- **Implementation planning** begins after the audit has established enough reuse, availability, ownership, and contract constraints to avoid material guesses.

These neighboring workflows are optional. The audit must still produce a useful result when they are unavailable.

## Stop before implementation

The audit is complete when every in-scope material capability is either classified with sufficient evidence or explicitly left `UNKNOWN`, and any material version/availability or external-contract gaps are separated from ownership.

Do not continue into file-by-file planning, code changes, Figma mutation, Code Connect authoring, package publication, backend contract design, or tracker lifecycle work. Return the established boundary to the workflow that owns those actions.
