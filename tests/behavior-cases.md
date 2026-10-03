# UI Reuse Audit behavioral cases

Use these cases to evaluate classification quality after the skill activates. Grade semantic outcomes, evidence use, and stop behavior rather than exact prose.

## Observable contract

A successful run should:

- classify each material capability with exactly one primary decision or `UNKNOWN` when evidence is insufficient;
- report availability separately from ownership;
- keep external contract state separate from UI ownership;
- prefer consumer-resolved evidence over upstream possibility;
- avoid duplicate local components, unnecessary wrappers, invented contracts, and repository-wide scans;
- return a compact audit handoff and stop before implementation.

## Primary decision cases

### 1. `REUSE` — existing primitive

Context:

- the application already imports `Button` from the current resolved design-system package;
- the new screen needs the same semantics and supported variants;
- no additional framework integration is required.

Expected observations:

- `Decision = REUSE`;
- `Availability = AVAILABLE`;
- required action is `reuse`;
- does not propose a local `PrimaryButton` wrapper or a design-system extension.

Failure mode caught: recreating an already available design-system primitive locally.

### 2. `REUSE` — existing adapter

Context:

- the design system exports a framework-neutral `TextLink`;
- the application already has a shared router-aware adapter around it;
- the new screen needs the same routing integration.

Expected observations:

- existing adapter is classified as `REUSE`, not `ADAPTER`;
- `Availability = AVAILABLE`;
- required action is `reuse`;
- no second wrapper is proposed.

Failure mode caught: wrapper proliferation around an existing integration boundary.

### 3. `ADAPTER` — new real integration boundary

Context:

- the design system exports a framework-neutral image/avatar primitive;
- the application framework requires its own runtime image component and optimization contract;
- no existing app adapter performs that translation;
- the translation is reusable across multiple features.

Expected observations:

- `Decision = ADAPTER`;
- ownership points to the application integration/shared adapter layer, not the design system primitive itself;
- required action is `add adapter`;
- does not push the framework runtime dependency into the generic design-system primitive.

### 4. `COMPOSE` — feature/domain assembly

Context:

- the design system already exports Card, Text, Badge, Button, and EmptyState;
- the requested `ProfileStations` block combines those primitives with profile-specific data, labels, grouping, and feature behavior;
- no new generic interaction primitive is missing.

Expected observations:

- `Decision = COMPOSE`;
- ownership is feature/domain layer;
- required action is `compose locally`;
- does not classify the whole block as a new design-system component merely because similar blocks may appear elsewhere in the app.

### 5. `EXTEND_DESIGN_SYSTEM` — missing reusable primitive

Context:

- multiple unrelated application surfaces need the same controlled dialog primitive;
- the required contract is application-independent: open state, close callback, title/content slots, actions, focus/escape behavior;
- current design-system package and upstream source do not provide it;
- application routes, services, analytics policy, and business state are not part of the primitive contract.

Expected observations:

- `Decision = EXTEND_DESIGN_SYSTEM`;
- `Availability = ABSENT` when representative evidence proves absence;
- required action is `extend design system`;
- application orchestration remains outside the primitive.

Failure mode caught: keeping a genuinely generic primitive local because the first use happens in one feature.

### 6. `APP_SPECIFIC` — domain-only behavior

Context:

- a widget coordinates domain-specific permissions, service calls, audit events, and product-specific wording;
- it uses shared design-system controls internally;
- extracting it would require exposing domain concepts through a fake generic API.

Expected observations:

- `Decision = APP_SPECIFIC`;
- ownership remains application/domain;
- required action is `keep app-specific`;
- shared primitives are still reused internally.

## Availability and version cases

### 7. Upstream component absent from current consumer version

Context:

- application lockfile resolves `@acme/ui@1.4.0`;
- installed/public exports for `1.4.0` do not contain `Popover`;
- upstream design-system `main` contains `Popover` but it is unreleased.

Expected observations:

- ownership decision remains `REUSE` because the reusable capability already exists upstream;
- `Availability = UPSTREAM_ONLY`, not `AVAILABLE`;
- required action is `publish/consume upstream capability` or equivalent;
- does not create a local popover or classify the gap as `EXTEND_DESIGN_SYSTEM`.

Failure mode caught: trusting design-system `main` as if it were already importable by the consumer.

### 8. Released newer version creates a version gap

Context:

- manifest declares `@acme/ui: ^1.4.0`;
- lockfile resolves `1.4.2`;
- `Popover` was introduced and publicly exported in released `1.6.0`;
- the application currently resolves `1.4.2`.

Expected observations:

- `Decision = REUSE`;
- `Availability = VERSION_GAP`;
- required action is `align/upgrade dependency`;
- exact resolved version `1.4.2` is used for current availability, not the manifest range alone.

### 9. Manifest range vs resolved-version distinction

Context:

- manifest says `@acme/ui: ^2.0.0`;
- lockfile and installed package resolve `2.1.3`;
- a required component exists in `2.1.0+` but not `2.0.0`.

Expected observations:

- current availability is based on resolved `2.1.3`;
- does not report a false `VERSION_GAP` by treating `^2.0.0` as exact `2.0.0`;
- records the declared range only as supporting dependency context.

## External contract cases

### 10. `MISSING_CONTRACT` — approved UI action has no route/API/action

Context:

- approved UI includes a “Create station” action;
- no route, navigation target, API mutation, or product-approved temporary behavior exists yet;
- the visual control is still part of the approved screen.

Expected observations:

- UI ownership is classified independently from the missing action contract;
- external contract state is `MISSING_CONTRACT`;
- does not invent `/stations/new`, a mutation signature, placeholder navigation, no-op callback, fake success behavior, or local-only substitute;
- audit records a concrete TODO and an honest disabled/unavailable state when appropriate;
- stops before designing the backend/route contract.

Failure mode caught: inventing missing application contracts to make the UI appear complete.

### 11. Contract state `UNKNOWN` without ownership impact

Context:

- the UI primitive and local composition ownership are fully established;
- documentation does not reveal whether the save action already exists elsewhere in the app;
- that uncertainty does not change where the UI belongs.

Expected observations:

- keeps the established UI decision;
- marks the external contract as `UNKNOWN`;
- required next step is limited to verifying the contract;
- does not turn the whole UI decision into `UNKNOWN` unnecessarily.

## Evidence and inconclusive cases

### 12. Insufficient evidence for availability

Context:

- manifest declares a UI dependency;
- lockfile is unavailable;
- installed package cannot be inspected;
- upstream repository contains a component with the expected name;
- no consumer import/call site proves current availability.

Expected observations:

- does not claim `AVAILABLE`;
- availability is `UNKNOWN` or `UPSTREAM_ONLY` only if upstream-only status is actually established;
- required action is `verify evidence` when the decisive consumer state is missing;
- does not infer the resolved version from the manifest range.

### 13. One failed search is not `ABSENT`

Context:

- searching one local barrel file does not find `DataEmptyState`;
- secondary entry points and the installed package exports have not been inspected.

Expected observations:

- does not assign `ABSENT` yet;
- continues only through the smallest representative checks needed;
- returns `UNKNOWN` if those checks cannot be performed.

## Figma evidence cases

### 14. Code Connect confirms intended counterpart but not availability

Context:

- the Figma component is mapped through Code Connect to `@acme/ui/NavigationBar`;
- current application package version has not yet been verified.

Expected observations:

- uses the mapping as strong evidence for intended reusable ownership;
- separately verifies consumer package availability;
- does not create/update Code Connect or mutate Figma;
- does not claim `AVAILABLE` from the mapping alone.

### 15. Visual similarity without component/library identity

Context:

- a Figma sidebar looks similar to an existing design-system navigation component;
- no library identity, Code Connect mapping, or matching semantic contract has been established.

Expected observations:

- does not conclude `REUSE` from appearance alone;
- inspects semantic contract and consumer evidence proportionally;
- preserves `UNKNOWN` or another evidence-backed decision rather than guessing from visuals.

## Proportionality and stop cases

### 16. One-component task stops after decisive evidence

Context:

- task changes one dropdown-like control;
- consumer already imports the exact shared Select primitive with the required behavior;
- no adapter or contract gap exists.

Expected observations:

- classifies `REUSE + AVAILABLE` from decisive consumer evidence;
- does not inspect unrelated buttons, icons, tokens, all design-system packages, or the upstream repository;
- returns one compact row and stops.

Failure mode caught: turning a focused audit into a repository-wide design-system inventory.

### 17. Mixed screen with one inconclusive capability

Context:

- screen contains three material capabilities;
- two are proven `REUSE + AVAILABLE`;
- the third may be an existing upstream primitive but package evidence is unavailable.

Expected observations:

- keeps the two established rows unchanged;
- marks only the third capability inconclusive with `verify evidence`;
- does not fail or reopen the entire audit;
- stops without implementation.

## Output-contract checks

For common cases, verify that the result can be represented by:

| UI element | Evidence | Availability | Decision | Required action | Ownership |
|---|---|---|---|---|---|

Assertions should target:

- one row per material capability;
- one primary decision or `UNKNOWN`;
- one availability status;
- concise traceable evidence;
- implementation-neutral required action;
- compact ownership layer/source;
- optional sections only for material design-system gaps, application boundaries, missing contracts, or verification notes.

Do not fail a test because headings, prose, or evidence ordering differ when the semantic contract is preserved.

## Execution protocol

1. Run each case in a fresh isolated conversation with the skill installed.
2. Provide the minimum synthetic repository/package/Figma facts named by the case; do not add hidden facts that make the classification easier.
3. Record observed `Decision`, `Availability`, external contract status where applicable, required action, scope expansion, and side effects.
4. Treat static markdown review as specification coverage, not behavioral execution.
5. If a case fails, refine the narrowest skill rule that explains the repeated failure and rerun that case plus the nearest regression cases.
6. Preserve execution records with model/version, date, observed result, and resulting skill change before publishing a release.