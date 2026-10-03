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

## Keep availability orthogonal to ownership

Ownership answers **where the capability belongs**. Availability answers **whether the current consumer can use the relevant capability now**. Do not collapse the two.

Use a compact availability status such as:

- `AVAILABLE` — evidence shows the current consumer can use the capability;
- `UPSTREAM_ONLY` — the capability exists in an upstream source but is not part of the consumer's current usable surface;
- `VERSION_GAP` — the capability belongs to the dependency/design system but the consumer is on a version/channel that does not expose it;
- `ABSENT` — representative evidence supports that the capability is not present in the relevant reusable surfaces;
- `UNKNOWN` — available evidence is insufficient to establish one of the above confidently.

A capability can therefore be, for example, `EXTEND_DESIGN_SYSTEM + ABSENT`, or `REUSE + AVAILABLE`. A component that exists only on design-system `main` is not `REUSE + AVAILABLE` for a consumer that cannot import it.

## Keep external contracts orthogonal to UI ownership

A route, API, mutation, action, navigation target, permission, or other non-UI dependency is not a sixth ownership decision.

Track external-contract state separately, for example:

- `AVAILABLE` — the required contract exists and is established for the task;
- `MISSING_CONTRACT` — the approved UI requires a contract that does not exist or is not yet provided;
- `UNKNOWN` — the evidence does not establish whether the contract exists.

Never invent a missing contract to make the UI appear complete. The UI ownership classification can still be established while the interaction remains blocked by `MISSING_CONTRACT` or `UNKNOWN`.

## Preserve the important distinctions

- **Consumer availability is not upstream existence.** Upstream source proves a capability exists somewhere, not that this consumer can use it.
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

Detailed consumer-first discovery order, exact evidence checks, and absence-proof requirements are defined by the dedicated discovery workflow. This section defines only the behavioral boundary that workflow must preserve.

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
