# ML Concepts, Models, and Algorithms to Know


1. Time-series & forecasting
- Question: What basics should engineers know (stationarity, seasonality, autocorrelation, differencing)?
- Answer: Engineers must recognise when a series requires detrending or differencing, identify seasonality windows, and use ACF/PACF to pick model orders. For Vigil, ensure selection chooses stable series for the clock.
- Has solution: YES — team familiarity expected; include quick-reference notes for demo.

-- Question: What forecasting methods are relevant (ARIMA, Exponential Smoothing, state-space, neural)? When to prefer each?
-- Answer: Use simple methods (ETS/ARIMA) for interpretable baselines and neural methods for complex, high-volume series. For the demo, start with robust classical methods; show calibration metrics for neural approaches if used.
-- Has solution: PARTIAL — current `clockd` implementation provides a deterministic, contract-conformant stub only. See `clockd/src/clockd/forecast.py` for `StubClock`/`stub_forecast`: it holds the last level as the point forecast and returns a widening uncertainty band (the line-never-collapses guarantee).

-- Enabled algorithms (current):
	- `StubClock` / `stub_forecast` (honest, pure-stdlib placeholder)

-- How to add or enable production algorithms:
	- Implement a replacement clock class that returns a `ForecastResult` (matching the shape in `clockd/src/clockd/forecast.py`).
	- Wire the implementation into `clockd/src/clockd/server.py` (serve via gRPC) and expose a selection mechanism (env var or config flag) so operators can pick the algorithm at runtime.
	- Add the model runtime as an optional `model` extra in `pyproject.toml` to keep base installs light.
	- Enforce the no-string-fields contract at the proto level and add a unit/conformance test parsing compiled descriptors to reject string fields.
	- Add harness backtests and calibration checks (coverage, CRPS/PIT) and gate production rollout behind these tests.

-- How to test new algorithms:
	- Unit tests for edge cases (empty series, very noisy series, presence/absence of covariates) and to assert band width never collapses.
	- Backtests in `harness/` comparing forecasts against held-out windows; include interval-coverage assertions.
	- CI: ensure `just ci` runs `buf lint`, conformance tests, and harness validation before accepting a new clock implementation.

-- Has solution status: STUB_ONLY — the repo includes the stub and the swap path, but production forecasting engines (TimesFM, ARIMA stacks, neural models) are not yet implemented/wired and require the steps above.

- Question: How to evaluate forecasts and why must uncertainty bands not collapse?
- Answer: Use MAPE/RMSE for point forecasts and interval coverage metrics for bands; enforce minimum variance to avoid overconfident bands—report coverage defects in harness tests.
- Has solution: PARTIAL — harness provides backtest hooks; tighten calibration checks.

2. Anomaly detection
- Question: Statistical thresholds vs unsupervised models — when to use which?
- Answer: Use deterministic/statistical thresholds for customer-declared limits and explainability; reserve unsupervised models for advisory detection with human-in-the-loop. Keep learned detectors out of gating detection decisions.
- Has solution: YES — design prescribes deterministic detection for gating.

- Question: How do `obsd` deterministic patterns differ from learned detectors?
- Answer: `obsd` uses fingerprints and topological matches with explicit provenance; learned detectors infer patterns and are flagged as PROJECTED or advisory only.
- Has solution: YES — provenance rules enforce separation.

3. Graphs and topology
- Question: Key graph concepts engineers should know?
- Answer: Entity modelling, authored edges, overlays, and release-based versioning. Know how binding overlays affect selection and fingerprinting.
- Has solution: YES — ontology folder and docs provide guidance.

- Question: How to safely propagate signals across topology?
- Answer: Use deterministic traversals and subtract graph-explained footprints before surfacing unexplained mass; surface as authored hypotheses, not causal claims.
- Has solution: YES — blueprint prescribes this behavior.

4. Causality vs correlation
- Question: How to educate stakeholders on co-occurrence vs causation?
- Answer: Present co-occurrence as evidence with provenance and require authored edges for causal claims; provide replay-based what-if simulations rather than causal assertions.
- Has solution: YES — policy explicitly forbids inferred causation.

5. Provenance and interpretability
- Question: Why immutable provenance classes matter?
- Answer: They preserve trust and auditability. Never collapse PROJECTED into MEASURED; show origin and uncertainty for each claim.
- Has solution: YES — core design; enforce in code reviews and tests.

6. Model risk and governance
- Question: How to limit model surface area and ensure governance?
- Answer: Restrict models to the clock process, require versioning, audit trails, and gating backtest metrics before production rollout. Keep model outputs separate and labelled.
- Has solution: PARTIAL — process exists; formal gating may need checklist enforcement.

7. Practical algorithms and tools
- Question: What practical streaming/time-series tooling matters?
- Answer: Understand windowing, aggregation, sampling, downsampling, and proto-based contracts for cross-language reproducibility. Familiarity with replay and harness tools is essential.
- Has solution: YES — docs and harness provide workflow.

8. Testing for ML systems
- Question: How to test ML/time-series components for determinism and correctness?
- Answer: Use golden-file fixtures, injected clocks, unit tests without network calls, and integration tests behind `//go:build integration`. Automate calibration checks in harness tests.
- Has solution: PARTIAL — harness exists; extend calibration and gating checks as needed.
