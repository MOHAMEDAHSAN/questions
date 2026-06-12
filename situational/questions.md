# Situational / Scenario-Based Questions

This file contains scenario-based questions you can use for mock interviews, rehearsals, and runbook checks. Each entry includes the question, a concise answer, and a short "Has solution" status that indicates whether our repo already supports the capability.

---

1) Incident handling

Question: Describe how `obsd` would detect an abrupt CPU cascade (see `chaos/cpu-cascade.yaml`). Which components produce the detection, and what are the operator-visible outputs?

Answer: Metrics are ingested by the `observe` layer. Fingerprints are computed and condensed into signatures. The `detect` stage matches fingerprints against authored and deterministic rules and emits findings. The operator sees the entity inventory, findings with provenance, and entries in the unexplained channel. To debug a missing finding: verify metric ingestion, confirm selection included the series, inspect fingerprint logs, and compare with replay/golden fixtures.

 
---

Question: Walk through the pipeline from raw metrics → fingerprint → detection → finding. How would you debug a missing finding for an obvious spike?

Answer: Verify raw metrics are present in the store, confirm selection chose the series, reproduce the fingerprint via `replay`, inspect detection logs and binding overlays, and ensure graph version/params/codegen all match for replay parity.

 
---

Question: If the clock (forecasting) is degraded or absent, which detections change and which remain deterministic? How do you communicate that to operators?

Answer: Deterministic detections (fingerprint/topology) remain active; PROJECTED detections that depend on forecasts will not fire. Surfacing should clearly label clock health and mark projected-only findings as inactive.

 
---

2) False positives and explainability

Question: A customer reports spurious detections during a deployment window. How would you triage whether the detections are measurement noise, binding gaps, or faulty thresholds?

Answer: Inspect deployment metadata, raw metric variance, binding coverage for affected entities, and declared thresholds. Run replay across pre/post-deploy windows to check persistence of detections.

 
---

Question: How do you present provenance to show why the system flagged an entity (MEASURED vs PROJECTED vs AUTHORED)?

Answer: Surface each statement with its provenance label and direct evidence: measured windows for MEASURED, forecast bands+horizon for PROJECTED, and author note/version for AUTHORED.

 
---

3) Scaling and topology changes

Question: How does the system handle sudden topology change (many pods recreated across namespaces)? How do joins and identity affect fingerprint continuity?

Answer: Identity attempts stable joins (UIDs, stable labels). During churn present decreased identity confidence and mark entities as transient. Debounce findings for short-lived entities and surface binding gaps.

 
---

Question: If the kind cluster/network is partitioned, what parts of the pipeline degrade gracefully and what fails hard?

Answer: Local deterministic detection can continue where local data exists; cross-cluster joins and global graph refreshes fail. Surfacing should indicate degraded components and show partial inventories.

 
---

4) Data gaps and backfills

Question: If historical series are missing for a newly added entity, how does selection and the clock decide whether to run forecasting or declare "unbounded"?

Answer: Selection checks data availability and declared bounds. If no history and no declared threshold, surface the variable as "unbounded". The clock may only forecast using related series when selection allows.

 
---

Question: Describe a recovery plan to replay missed observations and validate determinism (golden-file fixtures).

Answer: Ingest backfilled data, run `replay` with the same graph and params, compare outputs byte-for-byte against golden fixtures, and update fixtures only when changes are intentional and documented.

 
---

5) Operator actions

Question: What runbook steps should be exposed when the unexplained channel grows: acknowledge, annotate, mark false-positive, or tune thresholds?

Answer: Provide a triage flow: inspect provenance, annotate, temporarily suppress, update declared thresholds via config + staging replay, and record remediation. All actions must be auditable.

 
---

Question: If an operator updates a customer's declared thresholds, how and where should that update be validated and tested?

Answer: Validate changes via staging replay, apply to params/config store, run `just ci` if detection gating is affected, and preserve an audit trail.


