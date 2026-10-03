# v0.1.0 real-project validation

Date: 2026-10-03

## Result

**PASS** for the v0.1.0 decision contract on a real frontend/design-system fixture.

The audit changed the implementation boundary in the intended ways: it found existing reusable UI and adapters before proposing new local UI, refused to use design-system `main` as a proxy for the consumer package, separated a generic dialog ownership gap from profile-specific composition, preserved a missing route as `MISSING_CONTRACT`, and stopped at a compact planning handoff.

No Soniks source code or Figma content was mutated during this validation.

## Validation mode and limitation

This run applied the finished `ui-reuse-audit` contract semantically in a ChatGPT session using GPT-5.6 Sol with connected GitHub and Figma evidence.

It was **not** an isolated Codex CLI run and did not independently measure model-driven implicit skill routing. Implicit/negative activation routing is covered by `tests/activation-cases.md`; this report validates the audit's behavior once the skill is in scope against real project evidence. A trivial-edit negative control is recorded below without claiming an independent router execution.

## Real fixture

### Consumer application

- repository: `sonik-space/soniks-frontend`
- branch: `SD-SW-GS-1818-SONIKS-V2-develop-user-account-and-profile`
- branch HEAD during validation: `d35726d8c26034f1f86cc491bc8844a27d6d6679`
- task surface: account/public-profile UI and related shared UI/adapters

### Design system

- repository: `sonik-space/soniks-design-system`
- matching feature branch: `SD-SW-GS-1818-SONIKS-V2-develop-user-account-and-profile`
- branch HEAD during validation: `65800e9423c768c3df2370124b07f7862260e344`

### Figma evidence

- file key: `WnAosCmXUid8NQaYpcFDT0`
- inspected public node: `7061:87569`
- the inspected design context includes shared-looking navigation/UI elements and an `unauthorized-modal` frame.

Figma was used only as optional design/ownership evidence. No Code Connect mapping or Figma asset was created or modified, and Figma identity was not treated as proof of package availability.

## Evidence boundary

The audit deliberately did **not** inventory either repository. It inspected only the surfaces needed for the profile task and the validation questions:

- consumer `package.json`;
- design-system `packages/ui/package.json` on `main` and the matching feature branch;
- design-system public UI exports and `DataEmptyState`;
- frontend shared `TextLink` / locale-link integration;
- profile tabs, header/profile links, profile empty state, and new-station control;
- frontend shared modal primitives (`BaseModal`, `ModalBackdrop`, modal hooks);
- one relevant Figma node.

This was enough to establish the material ownership/availability boundaries without a repository-wide scan.

## Consumer-first version evidence

The consumer declares exact `@sonik-space/ui` version `0.1.2` in `soniks-frontend/package.json`.

The matching design-system feature branch identifies `packages/ui` as `0.1.2`, while design-system `main` identifies the package as `0.1.1`.

This is a real version-drift hazard and validates the rule that upstream `main` must not be used as the consumer's availability source. The audit used the consumer state first and consulted the matching design-system surface only as supporting evidence.

Importantly, the sampled public `packages/ui/src/index.ts` content was identical on `main` and the feature branch during this run. Therefore the audit **did not invent a `VERSION_GAP` for a component merely because the package versions differ**. A component-level `VERSION_GAP` requires evidence that the relevant released/resolved versions expose different capabilities.

That distinction is the desired behavior: version numbers establish that provenance must be checked; they do not by themselves prove a capability delta.

## Compact audit handoff

| UI element | Evidence | Availability | Decision | Required action | Ownership |
|---|---|---|---|---|---|
| Locale-aware / external text links | `src/shared/ui/text-link/TextLink.tsx` already adapts design-system `TextLink`; `ProfileLinks.tsx` consumes the shared adapter | `AVAILABLE` | `REUSE` | reuse | existing application adapter + design-system text primitive |
| Profile tab link integration | `ProfileTabs.tsx` uses design-system `CategoryTab` with existing `LocaleLinkRenderer`; no new wrapper is required | `AVAILABLE` | `REUSE` | reuse | design-system primitive + existing application routing adapter |
| Profile-specific tabs/content assembly | active segment, translated labels, profile paths and prefetch behavior belong to the profile feature | `AVAILABLE` | `COMPOSE` | compose locally | feature/domain layer |
| Generic empty-state base | design-system `DataEmptyState` provides generic icon/title/description/action-slot behavior | `AVAILABLE` | `REUSE` | reuse where its contract fits | design-system |
| Rich profile empty-state presentation | local `ProfileEmptyState` adds profile-specific secondary/accent text composition beyond the generic state contract | `AVAILABLE` | `COMPOSE` | compose locally | profile feature/domain layer |
| Generic modal mechanics | frontend already has generic `shared/lib/modals/BaseModal`, backdrop, escape/scroll-lock helpers; design-system public UI exports no dialog/modal primitive | `AVAILABLE` | `EXTEND_DESIGN_SYSTEM` | extend design system | shared design system; current implementation is mis-owned application-shared UI |
| Profile links modal content/orchestration | `ProfileLinks.tsx` owns link normalization, list content, open/close state and profile wording; these are not generic dialog mechanics | `AVAILABLE` | `COMPOSE` | compose locally | profile feature/domain layer |
| New-station visual control | `ProfileNewStationButton.tsx` uses the design-system `Button` | `AVAILABLE` | `REUSE` | reuse | design-system |
| New-station navigation/orchestration | creation/navigation belongs to the application and the route is not yet available | `AVAILABLE` | `APP_SPECIFIC` | resolve missing contract | application/domain |

## `ADAPTER` boundary check

No material capability in this selected profile task justifies a **new** adapter.

The two strongest framework-integration candidates already have application integration surfaces:

- shared `TextLink` injects `LocaleLinkRenderer` into the design-system primitive;
- profile tabs pass the existing `LocaleLinkRenderer` to `CategoryTab`.

The correct classification for those integrations is therefore `REUSE`, not `ADAPTER`. This is a useful real-project negative control: the audit prevents wrapper proliferation instead of manufacturing an `ADAPTER` row simply to exercise every enum value.

The positive `ADAPTER` case remains covered by `tests/behavior-cases.md`: `ADAPTER` is selected only when a reusable primitive exists and a **new real** environment/framework integration boundary is actually required.

## Design-system gap

The real project contains a generic `BaseModal` under application `shared/lib/modals` with only generic inputs (`children`, position, background-click handling) plus generic backdrop/body-scroll behavior. The design-system public UI surface inspected for this run does not export a dialog/modal primitive.

This is the clearest real fixture for `EXTEND_DESIGN_SYSTEM`: the generic mechanics are reusable and application-independent, but currently live in the application. The audit therefore prevents the opposite failure mode from simple reuse checks: keeping a genuinely reusable primitive application-local forever.

For the profile feature itself, `ProfileLinksModal` remains a feature composition. Moving generic modal mechanics upstream does **not** move profile link normalization, labels, state, or content into the design system.

## Missing contract

`ProfileNewStationButton.tsx` contains the concrete TODO:

> add navigation to station creation when the route appears

The control is rendered with the existing design-system `Button` and is disabled. No route string, action signature, fake success callback, console-only behavior, or placeholder navigation is invented.

Contract state: `MISSING_CONTRACT`.

This validates the orthogonal model:

- UI ownership: `REUSE` for the visual button, `APP_SPECIFIC` for application navigation/orchestration;
- consumer availability: the button is `AVAILABLE`;
- external contract: `MISSING_CONTRACT`.

The audit preserves the approved UI as far as possible and leaves the unavailable interaction honest.

## Figma evidence behavior

The inspected Figma node supports the existence of navigation and modal-like design intent, but the audit did not infer code availability from visual identity.

For example, seeing an `unauthorized-modal` frame is supporting evidence that dialog behavior is a material UI capability; it is **not** evidence that `@sonik-space/ui@0.1.2` exports a dialog. Package/export evidence remains authoritative for consumer availability.

No broad Figma library reconciliation was performed because it was not needed to decide these boundaries.

## Regression review

| Failure mode | Real-project observation | Validation result |
|---|---|---|
| Duplicate local component despite reusable UI-kit primitive | `TextLink`, `CategoryTab`, `Button`, and `DataEmptyState` are discovered before proposing replacements | PASS — reuse is selected first |
| Unnecessary wrapper around an existing adapter | `TextLink` and `LocaleLinkRenderer` already cover the routing/link boundary | PASS — existing adapter remains `REUSE`, not `ADAPTER` |
| Importing a component not present in the consumer version | consumer is `@sonik-space/ui@0.1.2` while DS `main` metadata is `0.1.1` | PASS — main is not trusted as consumer state; no unproven component delta is invented |
| Putting a domain-specific composition into the shared design system | profile tabs, profile links modal content, and rich profile empty-state content contain profile semantics | PASS — kept `COMPOSE`/feature-owned |
| Keeping a genuinely reusable primitive application-local | generic `BaseModal` and modal mechanics are currently application-shared and absent from DS public API | PASS — flagged `EXTEND_DESIGN_SYSTEM` without moving feature orchestration upstream |
| Inventing a nonexistent contract to make a control appear functional | station-creation route is absent; current control has TODO + disabled state | PASS — preserved as `MISSING_CONTRACT` |

## Planning behavior changed by the audit

A downstream planner receiving this handoff should not start by creating profile-specific replacements for shared primitives. The constrained plan is instead:

1. reuse existing design-system primitives and existing application adapters;
2. compose profile semantics locally;
3. reuse the current generic modal mechanics rather than duplicating another shell, while tracking the generic modal primitive as design-system-owned work;
4. keep station-creation orchestration application-owned and blocked by the missing route rather than guessing a contract;
5. evaluate package capability against the consumer's resolved/declared target, not design-system `main`.

The audit intentionally stops here. It does not select files to edit, design the future dialog API, publish a package, implement the route, or mutate Figma.

## Trivial-work activation control

Control task:

```text
Change the existing profile button label and do not change behavior or component structure.
```

Under the finished activation contract this is copy-only work with established ownership and therefore **out of scope** for `ui-reuse-audit`.

As noted above, this control validates the documented activation rule semantically; this run did not independently exercise Codex's implicit skill router.

## Skill-defect review

No new `SKILL.md` defect was found that requires a v0.1.0 rule change.

Two details were especially important and behaved correctly:

1. package-version mismatch alone did not become a fabricated `VERSION_GAP`;
2. the presence of an application-local generic primitive did not force `REUSE` as the final ownership answer — availability and ownership remained orthogonal, allowing `EXTEND_DESIGN_SYSTEM + AVAILABLE` for `BaseModal`.

No additional execution script is justified by this validation; the first release remains instruction-only.

## v0.1.0 acceptance check

- existing reusable components/adapters discovered before new local UI: **PASS**
- consumer package reality preferred over upstream `main`: **PASS**
- `REUSE`, `COMPOSE`, `EXTEND_DESIGN_SYSTEM`, and `APP_SPECIFIC` observed on real material capabilities: **PASS**
- `ADAPTER` correctly rejected where an adapter already exists; positive case remains covered by behavioral eval: **PASS**
- version state kept orthogonal and no unsupported `VERSION_GAP` inferred: **PASS**
- missing route kept as `MISSING_CONTRACT` without invention: **PASS**
- compact downstream-planning handoff produced: **PASS**
- audit stopped before implementation or external mutations: **PASS**
- trivial/mechanical work remains outside activation boundary: **PASS (semantic control; router not independently executed)**

Subject to the explicit routing-test limitation above, this real-project validation supports treating the repository as the initial usable `v0.1.0` baseline.