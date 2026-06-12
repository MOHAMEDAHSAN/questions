# Vigil — Basic Primer

What follows is a short, practical introduction to Vigil: what it is, how it works, common use cases, and scaling notes.

What is Vigil?
- A Kubernetes-native AI observability system that builds a versioned ontology graph (entities + relationships), runs a deterministic detection engine, and exposes a narrow forecasting clock. It surfaces: measured facts (telemetry-derived), authored graph facts, and short-term probabilistic projections.

How Vigil works (high level):
- Ingest: collect telemetry and metadata from clusters and workloads.
- Fingerprint: condense series/behaviour into deterministic signatures used for matching.
- Bind & select: use the ontology graph to bind telemetry to entities and select relevant series for analysis.
- Detect: run the deterministic detection engine to produce MEASURED statements (thresholds, matches, rates).
- Forecast (optional): a separate clock process produces PROJECTED forecasts with uncertainty bands.
- Surface: present findings, explanations (authored notes), and the unexplained channel for residuals requiring human review.

Key components (single-sentence):
- `obsd`: Go monolith implementing ingestion, selection, detection, and surfacing.
- `clockd`: separate forecasting process that emits probabilistic projections.
- `ontology/` & `proto/`: stores the schema, graph, and shared contracts (Protocol Buffers).
- `harness/` and `replay`: backtesting and deterministic replay for regression guarantees.

Common use cases:
- Cluster and application anomaly detection (noise-reduced, deterministic fingerprints).
- SLA/SLO monitoring and alerts tied to declared thresholds.
- Short-term forecasting for capacity planning or early-warning of threshold crossings.
- Root-cause affordances via authored graph explanations and selection-driven diagnostics.

Scalability & deployment notes:
- Design is modular: `obsd` is a single binary (modular monolith) intended to run in K8s; `clockd` is independently deployable for scaling forecasts.
- Scale by sharding selection workloads, using edge aggregation, and running multiple `obsd` replicas with leader-election for stateful writes.
- Determinism and replayability are core — use golden fixtures and injected clocks for tests to ensure identical outputs across runs.

Where to read next:
- `docs/00-master-system-design.md` — system blueprint.
- `docs/01-epistemic-separation-charter.md` — provenance rules.
- `clockd/` — forecasting implementation and examples.


