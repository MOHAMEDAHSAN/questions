# General Product, Deployment and Team Questions

High-level product, deployment, and team-readiness questions with concise answers and support status. Use this file for stakeholder Q&A and demo planning.

---

1) Product & value

Question: What customer problem does Vigil solve? Who are the main personas (SRE, Platform, Operator)?

Answer: Vigil surfaces unexplained operational phenomena by combining deterministic fingerprinting, topology-aware detection, and narrow forecasting. Primary personas are SREs (triage), Platform engineers (binding/identity), and Operators (actionable findings).

 
---

Question: What are the key success metrics you will show at the finale (precision/recall of detections, time-to-detect, false-positive rate, coverage)?

Answer: Present precision and recall on seeded incidents, median time-to-detect, false-positive rate over a rolling window, and coverage of declared variables. Tie metrics to replayed golden fixtures for credibility. When using SLO (Service Level Objective) and SLA (Service Level Agreement) terminology in slides, expand the acronyms for non-technical stakeholders.

 
---

2) Deployment and ops

Question: How is the system deployed (single binary `obsd`, `clockd` separate)? What are the resource and scale considerations?

Answer: `obsd` is a Go monolith for the deterministic pipeline; `clockd` runs separately for forecasting. Document CPU/memory needs and sharding strategies for large clusters.

 
---

Question: What runbooks and dashboards must be present for a demo (entity inventory, unexplained channel, binding coverage)?

Answer: Provide an entity inventory, unexplained channel view, binding coverage dashboard, clock health indicator, and a replay/golden-fixture runner. Include runbook steps for common failures.

 
---

3) Security and compliance

Question: How do you secure data in transit and at rest for metrics and graph data? Where are secrets stored and rotated?

Answer: Use TLS for transport, encrypt data at rest, and store secrets in a secrets manager (Kubernetes secrets, Vault) with rotation policies. Demo secrets should be ephemeral and rotated after use.

 
---

Question: What are the privacy considerations when surfacing entity metadata or human-authored notes?

Answer: Redact PII in the UI, store raw metadata encrypted with access controls, and require audited reveal actions. Implement opt-in/opt-out for sensitive fields.

 
---

4) Roadmap and trade-offs

Question: What features are out-of-scope for the demo but planned soon? Which ones would you de-prioritise and why?

Answer: Out-of-scope: learned causal inference and full auto-remediation. Prioritise deterministic detection, replay/golden-file reliability, and UX clarity; postpone automated causal attribution and heavy ML gating.

 
---

Question: What are the main technical risks (identity mis-joins, mis-calibrated bands, silent failure modes) and mitigations?

Answer: Mitigations include conservative joins with confidence labels, calibration checks for forecasts, explicit missing-series states, and rehearsed failure-mode runbooks.

 
---

5) Team readiness

Question: Who owns which component (identity, selection, clock, surfacing)? Who will answer deep technical questions during the finale?

Answer: Produce an owners matrix mapping components to responsible engineers and backups. Ensure each owner can run `just ci` (Continuous Integration), `just obsd`, and the replay harness.

 
---

Question: What rehearsals are recommended (failure-mode demos, replay deterministic tests, live cluster up/down)?

Answer: Rehearse `just up` + `just obsd`, run chaos scenarios in `chaos/`, replay golden fixtures, and run cold-start/restore tests. Record timings and failure explanations.

 
