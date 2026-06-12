# Advanced Situational Questions & Model Answers

1) Cross-tenant correlated cascade
- Question: Two tenants show near-simultaneous CPU spikes across shared nodes. How would the system determine whether this is a shared infra problem, coordinated workload change, or a correlated false-positive from instrumentation?
- Answer: Start with identity joins and topology: check entity-to-node mapping in the inventory rendering (`cmd/obsd/inventory.go`) to determine common hosts. Use fingerprints to see if the same metric dimensions and labels correlate. Check provenance: MEASURED evidence (raw metrics) vs AUTHORED bindings that link tenants to nodes. If correlation remains after removing explained graph footprint, escalate to operator with a finding showing co-occurrence and residual unexplained mass. Automated causal attribution is NOT allowed — we present co-occurrence + authored hypotheses.
- Has solution: YES — detection, fingerprinting, provenance and unexplained channel exist. Automated causality: NO (needs attention).

2) Silent failure / metric ingestion outage
- Question: The metric pipeline drops a series for several minutes (ingestion gap). How does the system detect silent failures and avoid false anomaly reporting on downstream gaps?
- Answer: The selection engine should mark series as degraded when continuity checks fail; `obsd/internal/selection` and selection tests control which series the clock runs on. Detections should treat missing windows as a special state (absent vs low). The replay deterministic tests (golden files) help to validate behavior. If we lack an explicit missing-series state in surfacing, this is a gap.
- Has solution: PARTIAL — detection pipeline and selection exist. Explicit operator-visible 'missing-series' surfacing: MAYBE/needs attention.

3) Topology churn and fingerprint continuity
- Question: During a rolling upgrade thousands of pods are recreated causing identity re-joins. How to preserve fingerprint continuity and not flood the unexplained channel?
- Answer: Use identity join heuristics in `obsd/internal/identity` to map old → new entity IDs (labels, stable identifiers). Surface binding coverage and declare transient binding gaps during churn. Throttle or debounce findings for entities with short-lived topology churn; use authored rules to suppress transient patterns during declared maintenance windows.
- Has solution: PARTIAL — identity join code exists; maintenance-window suppression and adaptive debounce require authored rules (some present, but policy surface may need improvements).

4) Forecast clock conflicts with deterministic path
- Question: If the clock produces a projected short-term spike that would trigger a projected detection while the deterministic engine sees no anomaly, how do you present both to avoid operator confusion?
- Answer: Keep provenance separate at surfacing: show the MEASURED detection status and the PROJECTED forecast with uncertainty band. Explain that deterministic path does not gate on forecast. Offer recommended action labels (monitor vs immediate action). Never convert PROJECTED into MEASURED.
- Has solution: YES — provenance separation and project/measured distinctions are core design rules.

5) Partial observability causing false equivalence
- Question: Two entities share a label but are actually distinct; the system joins them and produces a merged fingerprint causing mis-attribution. How to detect and fix?
- Answer: Detect unstable joins via identity confidence metrics; present operator the conflicting evidence (conflicting labels, differing metric patterns). Provide a remediation path: add authoritative authored relation (`AUTHORED`) or update binding rules to split identities. Add tests to prevent the same join rule from matching multiple distinct golden fixtures.
- Has solution: PARTIAL — identity join and authoritative overrides exist; automated detection of unstable joins may need attention.

6) Replay vs live-drift discrepancy debug
- Question: A replay using the same readings/golden fixtures produces different findings than a live run. How to diagnose quickly?
- Answer: Check: graph version (`ontology/releases`), proto/gen code version, topology snapshot, and clock injection state. Use `just gen-check` and `go test -race` and verify `proto/gen/` is identical. If the clock was injected differently (or had strings leaked), that can change outcomes. Ensure all inputs match: readings, graph, params file in `obsd/internal/params`.
- Has solution: YES — deterministic replay philosophy, golden files, and `just ci` exist; tooling to compare full input manifests should be run as part of the diagnosis.
