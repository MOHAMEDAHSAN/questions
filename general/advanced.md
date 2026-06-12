# Advanced General / Product & Ops Questions with Answers

1) SLOs and detection SLAs
- Question: What SLOs should Vigil expose (time-to-detect, false-positive rate, coverage), and how are they computed in multi-tenant environments?
- Answer: Define SLOs per-customer: median time-to-detect for seeded incidents, false-positive rate over a rolling window (normalized by explained mass), and coverage (= fraction of declared variables with valid selection). Compute metrics per-tenant and aggregate with weighted averages. Tie demo claims to verified backtests.
- Has solution: PARTIAL — metrics can be computed via replay/harness; per-tenant SLO dashboards may need implementation.

2) HA and disaster recovery for `obsd`
- Question: How do you run `obsd` in HA mode and recover from regional failures while preserving identity mappings and graph consistency?
- Answer: Run multiple `obsd` instances with leader election for write operations, shard read-only workloads. Persist graph releases and selection state in durable storage; ensure entity join keys are deterministic so a new instance can reconstruct joins. Use snapshot + WAL approach for fast recovery.
- Has solution: PARTIAL — single-binary design is present; HA patterns and stateful persistence strategies need engineering and ops playbooks.

3) Secrets and secure deployment
- Question: Where should API keys and secrets (metric endpoints, storage creds) live and how are they rotated for the demo and production?
- Answer: Use a secrets manager (Vault/K8s secrets) with short-lived tokens; CI and `just` tasks should not bake secrets. Provide a small runbook showing how to inject demo secrets for `just up` and rotate them post-demo.
- Has solution: NO / Needs attention — repo includes deployment manifests but not a documented secrets rotation/runbook path.

4) Compliance and data minimisation
- Question: How do you reduce PII in surfaced entity metadata while preserving useful troubleshooting context?
- Answer: Apply redact-first policies in surfacing layer: replace or hash sensitive identifiers, provide operator a reveal action after authenticated justification, and store raw metadata in encrypted long-term storage with audit logs.
- Has solution: PARTIAL — privacy considerations are documented; implementation of redaction and controlled reveal likely needs work.

5) Demo rehearsal scripts and failure-mode playbook
- Question: What exact scenarios should be rehearsed for the finale (list and run commands)?
- Answer: Rehearse: (a) `just up` + `just obsd` with the Online Boutique workload; (b) run `chaos/cpu-cascade.yaml` to show detection; (c) replay a golden fixture showing deterministic outputs; (d) simulate missing-series and show unexplained channel handling. Provide concrete commands and checklist.
- Has solution: YES — `just` recipes and chaos assets exist; produce a rehearsal checklist and scripts.

6) Team Q&A readiness matrix
- Question: Which team member answers which deep-dive topics during the finale (identity, selection, clock, harness, UI)?
- Answer: Build a matrix mapping components to owners; rehearse each owner answering 2–3 advanced questions from these files. Ensure at least one person can demonstrate `just ci` and `just obsd` locally.
- Has solution: NO / Needs attention — the repository contains component boundaries, but a human-assigned readiness matrix must be created.
