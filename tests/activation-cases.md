# UI Reuse Audit activation cases

Use these cases to evaluate whether `ui-reuse-audit` activates at the correct boundary. Test the skill as installed, without naming `$ui-reuse-audit` unless the case explicitly checks manual invocation.

## Observable contract

A successful activation run should:

- activate only when unresolved UI reuse, consumer availability, or ownership could materially change what gets built or where it belongs;
- inspect only the material UI capabilities exposed by the task;
- prefer current consumer evidence before upstream possibility;
- return the compact audit handoff and stop before implementation;
- avoid creating wrappers, design-system work, routes, APIs, actions, or tracker items as side effects;
- preserve `UNKNOWN` when missing evidence could change a material classification.

A successful non-activation run should continue with the requested workflow rather than inserting a reuse audit that cannot change the implementation boundary.

Do not grade exact headings or prose. Grade activation, scope, semantic classifications, evidence use, and side effects.

## Positive activation cases

### 1. Explicit pre-implementation reuse audit

```text
Before we implement this account page, audit whether the navigation, empty state, buttons, and confirmation dialog already exist in our UI kit or adapters. Tell me what should be reused and what actually belongs locally.
```

Expected observations:

- activates explicitly;
- limits the audit to the named material capabilities;
- distinguishes reusable ownership from current consumer availability;
- does not begin implementation or file-by-file planning.

### 2. Implicit Figma-to-code request with an existing design system

```text
Implement this approved Figma settings screen in our existing React application. We already use a shared design-system package and some framework adapters. The Figma frame contains a sidebar, text links, an empty-state panel, and a modal confirmation flow.
```

Expected observations:

- activates before implementation because the existing shared UI system can materially change the implementation boundary;
- treats Figma/library/Code Connect identity as optional reuse evidence when available, not as proof that the current package exports a component;
- checks consumer reuse/ownership before proposing local components;
- hands the result back to the implementation workflow rather than taking over Figma-to-code work.

### 3. Planning request with unresolved shared ownership

```text
Plan a new reusable filter panel for three product screens. We have shared inputs and buttons, but I am not sure whether the panel itself belongs in the design system, a framework adapter layer, or each feature.
```

Expected observations:

- activates because ownership is materially unresolved;
- distinguishes composition, adapter, design-system extension, and application-specific ownership rather than treating all reuse as one category;
- stops after the reuse/ownership boundary is established so downstream planning can continue.

### 4. Existing component may be hidden behind a consumer version gap

```text
We need a generic popover for a new screen. I found one on the design-system repository's main branch, but I do not know whether this application can import it from the package version it currently resolves. Audit before we build a local one.
```

Expected observations:

- activates because current consumer availability is uncertain and could prevent duplicate implementation;
- verifies consumer/resolved package state before treating upstream existence as available;
- preserves reusable ownership if the capability already exists upstream while reporting an availability gap separately.

## Negative activation cases

### 1. Trivial copy change

```text
Change the button label from “Create” to “Add item”. Do not change behavior or component structure.
```

Expected observation: does not activate; this is a copy-only edit with established ownership.

### 2. Isolated CSS adjustment with established ownership

```text
The existing shared Card is already the approved component. Reduce the margin below it from 24px to 16px in this feature stylesheet only.
```

Expected observation: does not activate; no material reuse or ownership question is exposed.

### 3. Greenfield visual ideation without a reusable system

```text
Sketch three visual directions for a brand-new landing page. There is no existing component library, shared UI layer, or codebase constraint yet.
```

Expected observation: does not activate; this is greenfield visual ideation without an existing reusable system to audit.

### 4. Reuse and ownership already settled

```text
Implement the approved plan exactly as written: use the existing shared DataTable and its existing router adapter. Keep the feature toolbar local. No new shared components are needed.
```

Expected observation: does not re-audit solely because shared UI is mentioned; follows the implementation workflow unless new evidence exposes a material gap.

### 5. Mechanical refactor

```text
Rename the local `ProfilePanel` component to `AccountPanel` and update imports without changing behavior or ownership.
```

Expected observation: does not activate; this is a mechanical change.

### 6. Non-UI task

```text
Fix the server cache key collision in this data loader and update its unit tests.
```

Expected observation: does not activate; the task is outside the UI reuse/ownership boundary.

## Contextual re-entry case

### 1. Settled implementation exposes a new material gap

```text
The approved implementation says to use `SharedDialog`, but during implementation we discover that the application-resolved package does not export `SharedDialog` while the design-system main branch does. Recheck only this part of the plan.
```

Expected observations:

- activates for the newly exposed material gap even though the broader implementation was already settled;
- audits only the affected dialog capability;
- does not reopen unrelated ownership decisions;
- distinguishes `REUSE` ownership from `UPSTREAM_ONLY` or `VERSION_GAP` availability when supported by evidence.

## Explicit invocation smoke

```text
$ui-reuse-audit Audit the reuse and ownership boundary for this UI task before I write the implementation plan.
```

Expected observations:

- activates explicitly;
- requests or inspects only the minimum task/code evidence required to classify material capabilities;
- returns the compact audit contract and stops before implementation.

## Execution protocol

1. Install the skill in an isolated profile with implicit invocation enabled.
2. Start each case in a fresh conversation and provide only the prompt plus the minimum artifacts it names.
3. For implicit activation cases, do not name the skill. Run the explicit invocation smoke separately.
4. Record whether the skill activated, whether it widened scope, which material capabilities it audited, and whether it caused side effects.
5. For negative cases, record whether the requested workflow proceeded without an unnecessary audit.
6. Repeat ambiguous routing cases before changing the skill description based on one result.

Static validation is necessary but does not count as an activation pass. Execution records should include model/version, date, observed behavior, and any resulting skill change.