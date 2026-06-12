# ML Concepts, Models, and Algorithms to Know

This file lists essential ML and time-series concepts with concise Q/A.

---

1) Time-series & forecasting

Question: What basics should engineers know (stationarity, seasonality, autocorrelation, differencing)?

Answer: Engineers should recognise when to detrend or difference a series, identify seasonality windows, and use ACF/PACF to inform model orders. Ensure selection picks stable series for the clock.

---
Question: What forecasting methods are relevant (ARIMA, ETS, state-space, neural)? When to prefer each?

Answer: Use interpretable classical methods for baselines and explainability:

- ARIMA (Autoregressive Integrated Moving Average): good for stationary or differenced series with clear autoregressive structure.
- ETS (Error/Trend/Seasonality exponential smoothing): robust for many business time-series with trend/seasonality and easy to explain.
- State-space / Kalman filters: useful for irregular sampling and when modelling latent states.
- Prophet-like models: easy calendar/holiday handling and quick baselines for business metrics.
- Neural/global models (LSTM, Transformer, N-BEATS): preferred at scale when many related series share patterns; require strong calibration and monitoring.

When to prefer: start with ETS/ARIMA for small sets where interpretability matters; scale to global/neural models when patterns are shared across many series and classical methods underperform.


Enabled algorithms (current):

- `StubClock` / `stub_forecast` — an honest, pure-stdlib placeholder that holds the last level as the point forecast and returns widening quantile bands (the "line-never-collapses" guarantee).

How to add or enable production algorithms:

- Implement a replacement clock class that returns a `ForecastResult` with the same wire shape.
- Wire the implementation into `clockd/src/clockd/server.py` and expose selection via config or an environment variable.
- Add the model runtime as an optional `model` extra in `pyproject.toml` to keep base installs light.
- Enforce the no-string-fields contract at the proto level and add conformance tests.
- Add harness backtests and calibration gates (coverage tests, Continuous Ranked Probability Score — CRPS, and Probability Integral Transform — PIT checks) before production rollout.

How to test new algorithms:

- Unit tests for edge cases (empty series, noisy series, covariate handling) and to assert band width never collapses.
- Backtests in `harness/` comparing forecasts to held-out windows with interval-coverage assertions.
- CI: ensure `just ci` runs `buf lint`, conformance tests, and harness validation.

Has solution status: STUB_ONLY — the repo provides the stub and a swap path; production clocks require integration work.

---

3) ML models (expanded)

This section lists practical models and short notes on when they are appropriate. Expand this before the finale as needed.

- ARIMA (Autoregressive Integrated Moving Average): classic univariate model; good for single-series forecasting when stationarity (or differencing) is achievable.
- ETS (Error/Trend/Seasonality): exponential smoothing families for level/seasonal decomposition.
- State-space / Kalman Filter: for latent-state modelling and irregular observations.
- Prophet (additive regression with holiday effects): easy-to-use business-oriented baseline.
- LSTM (Long Short-Term Memory networks): RNN-based model for sequence learning; use when temporal dependencies are complex and data is abundant.
- Transformer-based models (e.g., Informer, Autoformer): scalable for long-range dependencies and large collections of series.
- N-BEATS / DeepAR / DeepState: neural architectures tailored to forecasting tasks (global models that can transfer patterns).
- Isolation Forest / Local Outlier Factor (LOF): unsupervised anomaly detection methods for point anomalies.
- Autoencoders (reconstruction-based): useful for multivariate anomaly detection when reconstruction error indicates deviation.
- EWMA (Exponentially Weighted Moving Average): fast, interpretable smoothing/anomaly baseline for low-latency detection.
- Change-point detection (CUSUM, Bayesian): detect sudden structural breaks.

Guidance: For the demo, favour explainable baselines (ETS/ARIMA/EWMA) and show calibration metrics before demonstrating any neural models.


---

2) Anomaly detection

Question: Statistical thresholds vs unsupervised models — when to use which?

Answer: Use deterministic thresholds for declared customer limits and explainability; reserve unsupervised models for advisory detection with human review. Avoid learned models in gating decisions.

 

---

Question: How do `obsd` deterministic patterns differ from learned detectors?

Answer: `obsd` uses fingerprints and topological matches with explicit provenance; learned detectors infer patterns and should be surfaced as PROJECTED or advisory.

 

---

3) Graphs and topology

Question: What key graph concepts should engineers know?

Answer: Entity modelling, authored edges, overlays, and release-based versioning. Know how binding overlays affect selection and fingerprinting.

 

---

4) Causality vs correlation

Question: How to educate stakeholders on co-occurrence vs causation?

Answer: Present co-occurrence as evidence with provenance and require authored edges for causal claims. Provide replay-based scenario simulations, never present inferred causation as fact.

 

---

5) Provenance and interpretability

Question: Why do immutable provenance classes matter?

Answer: They preserve trust and auditability. Never collapse PROJECTED into MEASURED; always show origin and uncertainty for each claim.

 

---

6) Model risk and governance

Question: How to limit model surface area and ensure governance?

Answer: Restrict models to `clockd`, require versioning and audit trails, run gating backtests before rollout, and label model outputs clearly.

 

---

7) Practical algorithms and tools

Question: What streaming/time-series tooling matters?

Answer: Understand windowing, aggregation, sampling, downsampling, and proto-based contracts for reproducibility. Familiarity with replay and the harness is essential.

 

---

8) Testing for ML systems

Question: How to test ML/time-series components for determinism and correctness?

Answer: Use golden-file fixtures, injected clocks, unit tests without network I/O, and integration tests behind `//go:build integration`. Automate calibration checks in harness tests.

 
