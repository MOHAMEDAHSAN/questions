# Situational / Scenario-Based Questions

1. Incident handling
- Question: Describe how `obsd` would detect an abrupt CPU cascade (see `chaos/cpu-cascade.yaml`). Which components produce the detection, and what are the operator-visible outputs?
- Answer: `observe` ingests metrics; `fingerprint` computation emits condensed signatures; `detect` matches fingerprints against authored and deterministic rules producing findings rendered by `cmd/obsd` (inventory + findings renderers). Operator view: entity inventory, finding with provenance, and unexplained channel entries. To debug a missing finding: verify metric ingestion, confirm selection included the series, examine fingerprint logs, compare replay golden fixtures. 
- Has solution: YES — pipeline components exist; add step-by-step diagnostic checklists to runbooks.

- Question: Walk through the pipeline from raw metrics → fingerprint → detection → finding. How would you debug a missing finding for an obvious spike?
- Answer: Validate raw metric presence (store), verify selection chose the series, reproduce fingerprint via replay, inspect detection logs and binding overlays. If any input (graph version, params, codegen) differs, replay may diverge.
- Has solution: YES — replay and golden fixtures are the primary tool.

- Question: If the clock (forecasting) is degraded or absent, which detections change and which remain deterministic? How do you communicate that to operators?
- Answer: Deterministic detections (fingerprint/topology) remain; PROJECTED detections relying on forecasts won't fire. Surfacing must label absent/degraded clock state and mark projected-only findings as inactive. 
- Has solution: YES — design preserves determinism; ensure UI shows clock health prominently.

2. False positives and explainability
- Question: A customer reports spurious detections during a deployment window. How would you triage whether the detections are measurement noise, binding gaps, or faulty thresholds?
- Answer: Check deployment window metadata, inspect raw metric variance, verify binding coverage for affected entities, and check declared thresholds. Use replay with pre/post-deploy windows to see whether detections persist. 
- Has solution: PARTIAL — tooling for bindings and replay exists; integrate a deployment-aware filter in runbooks.

- Question: How do you present provenance to show why the system flagged an entity (MEASURED vs PROJECTED vs AUTHORED)?
- Answer: Surface each statement with its provenance label and the immediate evidence: measured windows for MEASURED, forecast band + horizon for PROJECTED, and author note/version for AUTHORED. 
- Has solution: YES — provenance-first surfacing is in the blueprint; ensure renderer consistency.

3. Scaling and topology changes
- Question: How does the system handle a sudden topology change (many pods recreated across namespaces)? How do joins and identity affect fingerprint continuity?
- Answer: Identity module attempts stable joins (UIDs, labels); during churn, present decreased identity confidence and mark entities as transient. Debounce findings for short-lived entities and surface binding gaps. 
- Has solution: PARTIAL — identity joins exist; policy for debouncing and transient suppression may need enhancement.

- Question: If the kind cluster/network is partitioned, what parts of the pipeline degrade gracefully and what fails hard?
- Answer: Local deterministic detection can continue if data is available; cross-cluster joins and global graph refreshes fail. Surfacing should mark degraded components and produce partial inventories. 
- Has solution: PARTIAL — architecture tolerates partial failures but runbook documentation needed.

4. Data gaps and backfills
- Question: If historical series are missing for a newly added entity, how does selection and the clock decide whether to run forecasting or declare "unbounded"?
- Answer: Selection checks availability and declared bounds; if no history and no declared threshold, declare unbounded and surface that limitation. The clock may run on related series only if permitted by selection. 
- Has solution: YES/PARTIAL — rules exist to prefer declared thresholds; explicit unbounded surfacing should be enforced in UI.

- Question: Describe a recovery plan to replay missed observations and validate determinism (golden-file fixtures).
- Answer: Ingest backfilled data to the store, run `replay` with the same graph and params, compare byte-identical outputs to golden fixtures, and update fixtures if change is intentional. 
- Has solution: YES — golden fixtures and replay are part of the harness.

5. Operator actions
- Question: What runbook steps should be exposed when the unexplained channel grows: acknowledge, annotate, mark as false-positive, or tune thresholds?
- Answer: Provide operator actions in order: triage (examine provenance), annotate, suppress (temporary), tune thresholds (via declared config), and record remediation. Each action must be auditable.
- Has solution: PARTIAL — UI actions exist conceptually; implement audited operator actions if not present.

- Question: If an operator updates a customer's declared thresholds, how and where should that update be validated and tested?
- Answer: Changes should be validated in a staging replay, applied to params/config store, and run through `just ci` tests if they affect detection gating. Maintain an audit trail.
- Has solution: PARTIAL — policy exists; automation for staging validation may need more work.
