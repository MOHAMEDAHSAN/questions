# Advanced ML Concepts & Algorithms Questions with Answers

1) Probabilistic forecasting and coherent uncertainty
- Question: How do you ensure forecast coherence and non-collapsing uncertainty bands across hierarchical series (pod → service → namespace)?
- Answer: Use hierarchical probabilistic forecasting (bottom-up or reconciliation methods) and propagate predictive distributions, not point forecasts. Ensure forecast bands are calibrated via backtests and never collapse by enforcing minimum variance floor based on sampling error. Evaluate with proper scoring rules (CRPS) and PIT histograms.
- Has solution: PARTIAL — `clockd` provides a forecasting path; hierarchical reconciliation and calibrated band enforcement may need additions.

2) Backtesting and falsification for probabilistic clocks
- Question: How do you validate a probabilistic forecast engine (TimesFM or others) so that its uncertainty is trustworthy for operator decisions?
- Answer: Run systematic backtests over held-out windows, compute coverage rates for declared intervals, use bootstrapped residual analysis, and measure whether uncertainty bands maintain prescribed coverage. Integrate these metrics into harness tests (`harness/`) and require a gating threshold for production.
- Has solution: PARTIAL — harness exists for backtests; tighten calibration checks and gating rules as needed.

3) Graph-structured signal propagation algorithms
- Question: How do you propagate anomaly signals on an attributed graph without inventing causal claims? What algorithms are safe to use?
- Answer: Use deterministic graph traversals to mark neighbourhoods and compute explained footprint subtraction. For scoring, use conservative heuristics (co-occurrence counts, topological distance) and expose them as authored hypothesis edges. Avoid learning edge weights that claim causality without human authoring.
- Has solution: YES — design enforces authored edges and deterministic propagation; implementational coverage should be audited.

4) Counterfactual / causal testing (what-if)
- Question: Operators may ask "If I drain node X, what will happen to fingerprint Y?" Can the system provide actionable counterfactuals?
- Answer: True causal counterfactuals require interventional models; the repo's policy forbids learned causation. We can provide scenario simulation by replaying historical patterns with synthetic edits (replay-based what-if) and surface uncertainty; but do not present them as causal proofs.
- Has solution: PARTIAL — replay-based simulation can be implemented, but causal modeling is out-of-scope (needs attention if required).

5) Drift detection for selection and fingerprint features
- Question: How to detect when selection heuristics or fingerprint features drift (e.g., metric label semantics change)?
- Answer: Monitor distributional shifts of key features, track mismatch rates for golden fixtures, and add alarms when fingerprint similarity distributions shift beyond thresholds. Surface these as AUTHORED alerts requiring human review.
- Has solution: PARTIAL — monitoring hooks exist but explicit drift-detection alerts may need building.

6) Explainability for complex detectors (graph + forecast hybrid)
- Question: If a detector combines topology, fingerprints and a projection to flag an entity, how do you produce a concise, truthful explanation for an operator?
- Answer: Produce a layered explanation: (a) raw evidence (MEASURED windows), (b) deterministic match (fingerprint/topology) with matching metrics, (c) projection summary with band and confidence, and (d) authored note recommending actions. Keep provenance labels visible and never present model outputs as facts.
- Has solution: YES — provenance-first surfacing is in the blueprint; ensure UI/renderer consistently shows layers.
