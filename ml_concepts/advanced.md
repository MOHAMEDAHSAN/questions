# Advanced ML Concepts & Algorithms Questions with Answers

Use these prompts to probe deep ML/system design understanding. Each entry contains a concise model answer and a "Has solution" status.

---

1) Probabilistic forecasting and coherent uncertainty

Question: How do you ensure forecast coherence and non-collapsing uncertainty bands across hierarchical series (pod → service → namespace)?

Answer: Use hierarchical probabilistic forecasting and reconciliation (bottom-up or top-down), propagate predictive distributions (not just points), and enforce a minimum variance floor so bands never collapse. Validate with CRPS and PIT histograms.

 
---

2) Backtesting and falsification for probabilistic clocks

Question: How do you validate a probabilistic forecast engine so its uncertainty is trustworthy for operator decisions?

Answer: Run systematic backtests on held-out windows, compute empirical coverage for declared intervals, use bootstrapped residuals, and surface calibration metrics in harness tests. Gate production rollout on meeting calibration thresholds.

 
---

3) Graph-structured signal propagation algorithms

Question: How do you propagate anomaly signals on an attributed graph without inventing causal claims? What algorithms are safe to use?

Answer: Use deterministic traversals to mark neighborhoods, subtract graph-explained footprints, and compute conservative scores (co-occurrence counts, topological distance). Expose propagation results as authored hypotheses, not learned causal edges.

 
---

4) Counterfactual / causal testing (what-if)

Question: Operators may ask "If I drain node X, what will happen to fingerprint Y?" Can the system provide actionable counterfactuals?

Answer: True causal counterfactuals require interventional causal models (out-of-scope). Provide replay-based scenario simulations with synthetic edits and surface uncertainty, but do not present them as causal proofs.

 
---

5) Drift detection for selection and fingerprint features

Question: How to detect when selection heuristics or fingerprint features drift (e.g., metric label semantics change)?

Answer: Monitor feature distributions, track golden-fixture mismatch rates, and surface alerts when similarity distributions cross thresholds. Require human review before changing selection rules.

 
---

6) Explainability for complex detectors (graph + forecast hybrid)

Question: If a detector combines topology, fingerprints and a projection to flag an entity, how do you produce a concise, truthful explanation for an operator?

Answer: Produce a layered explanation: raw MEASURED evidence, deterministic fingerprint/topology match, projection summary with band and confidence, and an authored recommendation. Always show provenance and uncertainty.

 
