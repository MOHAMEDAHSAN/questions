# General Product, Deployment and Team Questions


1. Product & value
- Question: What customer problem does Vigil solve? Who are the main personas (SRE, Platform, Operator)?
- Answer: Vigil surface unexplained operational phenomena by combining deterministic fingerprinting, topology-aware detection, and narrow forecasting. Personas: SREs (triage), Platform engineers (binding/identity), and Operators (actionable findings).
- Has solution: YES — product charter and demos target these personas.

- Question: What are the key success metrics you will show at the finale (precision/recall of detections, time-to-detect, false-positive rate, coverage)?
- Answer: Show: precision/recall on seeded incidents, median time-to-detect, false-positive rate over a rolling window, and coverage of declared variables. Tie to replayed golden fixtures for credibility.
- Has solution: PARTIAL — metrics are definable via harness; dashboards may need wiring.

2. Deployment and ops
- Question: How is the system deployed (single binary `obsd`, `clockd` separate)? What are the resource and scale considerations?
- Answer: `obsd` is a Go monolith for deterministic pipeline; `clockd` runs separately for forecasting. Resource docs should include CPU/memory needs and sharding strategy for large clusters.
- Has solution: YES/PARTIAL — deployment manifests exist; capacity planning documentation may need fleshing out.

- Question: What runbooks and dashboards must be present for a demo (entity inventory, unexplained channel, binding coverage)?
- Answer: Provide an entity inventory, unexplained channel view, binding coverage dashboard, clock health, and replay/golden-fixture runner. Include simple runbook steps for common failures.
- Has solution: PARTIAL — renderers exist; dashboard composition and runbooks should be finalized.

3. Security and compliance
- Question: How do you secure data in transit and at rest for metrics and graph data? Where are secrets stored and rotated?
- Answer: Use TLS for transport, encrypt at rest in storage, and keep secrets in a secrets manager (K8s secrets or Vault) with rotation procedures. Demo secrets should be ephemeral.
- Has solution: PARTIAL — manifests exist; secrets/runbook automation needs work.

- Question: What are the privacy considerations when surfacing entity metadata or human-authored notes?
- Answer: Redact PII in the UI, store raw metadata encrypted with access controls, and require audit logs for reveal actions. Include opt-in/opt-out policies for sensitive fields.
- Has solution: PARTIAL — policy exists; redaction/reveal mechanics require implementation.

4. Roadmap and trade-offs
- Question: What features are out-of-scope for the demo but planned soon? Which ones would you de-prioritise and why?
- Answer: Out-of-scope: learned causal inference, full auto-remediation. Prioritise deterministic detection, replay/golden-file reliability, and UX clarity; postpone automated causal attribution and heavy ML-driven gating.
- Has solution: YES — roadmap docs exist; confirm priority list before finale.

- Question: What are the main technical risks (identity mis-joins, mis-calibrated bands, silent failure modes) and mitigations?
- Answer: Mitigations: conservative joins with confidence labels, calibration checks for forecasts, explicit missing-series states, and rehearsed failure-mode runbooks.
- Has solution: PARTIAL — mitigations are documented; engineering work remains.

5. Team readiness
- Question: Who owns which component (identity, selection, clock, surfacing)? Who will answer deep technical questions during the finale?
- Answer: Create an owners matrix mapping each component to an owner and backup. Ensure each owner can run `just ci`, `just obsd`, and the replay harness.
- Has solution: NO — human assignments are outside repository; produce a readiness matrix as a next step.

- Question: What rehearsals are recommended (failure-mode demos, replay deterministic tests, live cluster up/down)?
- Answer: Rehearse `just up` + `just obsd`, run chaos scenarios from `chaos/`, run replay/golden fixtures, and run cold-start/restore tests. Record timings and failure explanations.
- Has solution: YES — artifacts and scripts are available; make a checklist for rehearsals.
