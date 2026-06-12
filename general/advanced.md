# Advanced General / Product & Ops Questions with Answers

Advanced product, ops and readiness questions with answers and support status. Use these for stakeholder prep and finale runbooks.

---


1) SLOs and detection SLAs

Question: What SLOs should Vigil expose (time-to-detect, false-positive rate, coverage), and how are they computed in multi-tenant environments?

Answer: Define per-customer SLOs (Service Level Objectives): median time-to-detect for seeded incidents, false-positive rate over a rolling window normalized by explained mass, and coverage (fraction of declared variables with valid selection). Compute per-tenant metrics and aggregate with weighted averages. Tie claims to verified backtests. When presenting externally, expand SLO and SLA (Service Level Agreement) acronyms.

 
---


2) HA and disaster recovery for `obsd`

Question: How do you run `obsd` in HA mode and recover from regional failures while preserving identity mappings and graph consistency?

Answer: Run multiple `obsd` instances with leader election for writes and shard read workloads. Persist graph releases and selection state in durable storage; use deterministic join keys so a new instance can reconstruct joins. Use snapshot + WAL for fast recovery. Note: HA stands for High Availability.

 
---

3) Secrets and secure deployment

Question: Where should API keys and secrets (metric endpoints, storage creds) live and how are they rotated for the demo and production?

Answer: Store secrets in a secrets manager (Vault or Kubernetes secrets) and use short-lived tokens. Do not bake secrets into CI or `just` recipes. Provide a runbook to inject demo secrets and rotate them after the event.

 
---

4) Compliance and data minimisation


Question: How do you reduce PII in surfaced entity metadata while preserving useful troubleshooting context?

Answer: Redact or hash sensitive identifiers in the UI (user interface), store raw metadata encrypted with access controls, and expose a controlled reveal action with audit logs. Provide opt-in/opt-out for sensitive fields. PII refers to Personally Identifiable Information.

 
---

5) Demo rehearsal scripts and failure-mode playbook

Question: What exact scenarios should be rehearsed for the finale (list and run commands)?

Answer: Rehearse: `just up` + `just obsd` with Online Boutique; run `chaos/cpu-cascade.yaml`; replay a golden fixture; simulate missing-series and demonstrate unexplained channel handling. Produce a checklist and scripts with commands.

 
---

6) Team Q&A readiness matrix

Question: Which team member answers which deep-dive topics during the finale (identity, selection, clock, surfacing)?

Answer: Create a readiness matrix mapping each component to an owner and backup. Rehearse each owner on 2–3 advanced questions and ensure at least one person can run `just ci` (Continuous Integration) and `just obsd`.

 
