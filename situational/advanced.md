# Advanced Situational Questions & Model Answers

Advanced scenario prompts and concise model answers for deep technical Q&A. Each entry states whether our current repo provides the capability or whether engineering attention is needed.

---

1) Cross-tenant correlated cascade

Question: Two tenants show near-simultaneous CPU spikes across shared nodes. How would the system determine whether this is a shared infra problem, coordinated workload change, or a correlated false-positive from instrumentation?

Answer: Start with identity joins and topology: check entity-to-node mapping in the inventory rendering to identify shared hosts. Compare fingerprints and metric dimensions to confirm correlation. Use provenance to separate MEASURED evidence from AUTHORED bindings. After subtracting graph-explained footprints, surface co-occurrence and residual unexplained mass to operators. Automated causality is not permitted; present hypotheses as authored notes.

 
---

2) Silent failure / metric ingestion outage

Question: The metric pipeline drops a series for several minutes (ingestion gap). How does the system detect silent failures and avoid false anomaly reporting on downstream gaps?

Answer: Selection should mark degraded series when continuity checks fail. Detections treat missing windows as a special state (absent vs low). Use replay with golden fixtures to validate behavior. If missing-series surfacing is absent, add operator-visible indicators.

 
---

3) Topology churn and fingerprint continuity

Question: During a rolling upgrade thousands of pods are recreated causing identity re-joins. How to preserve fingerprint continuity and not flood the unexplained channel?

Answer: Rely on identity join heuristics (UIDs, stable labels) to map old→new entity IDs. Surface binding coverage and declare transient gaps during churn. Debounce findings for short-lived entities and apply maintenance-window suppression where appropriate.

 
---

4) Forecast clock conflicts with deterministic path

Question: If the clock produces a projected short-term spike that would trigger a projected detection while the deterministic engine sees no anomaly, how do you present both to avoid operator confusion?

Answer: Keep provenance separate in surfacing: show MEASURED detection status and the PROJECTED forecast with its uncertainty band. Clarify that deterministic detections do not gate on projections and provide suggested actions (monitor vs immediate action).

 
---

5) Partial observability causing false equivalence

Question: Two entities share a label but are actually distinct; the system joins them and produces a merged fingerprint causing mis-attribution. How to detect and fix?

Answer: Detect unstable joins via identity confidence metrics and surface conflicting evidence to operators. Provide remediation via authored relations (`AUTHORED`) or updated binding rules, and add tests to prevent regressions.

 
---

6) Replay vs live-drift discrepancy debug

Question: A replay using the same readings/golden fixtures produces different findings than a live run. How to diagnose quickly?

Answer: Verify graph release, compiled proto/gen, topology snapshot, and clock injection. Run `just gen-check` and `go test -race`. Confirm all inputs (readings, graph, params) match for deterministic parity.

 
