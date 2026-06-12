# Advanced Specific (Architecture / Design) Questions & Answers

1) Deterministic replay under graph evolution
- Question: How do you guarantee byte-identical deterministic outputs when the ontology graph changes between runs? What versioning and compatibility constraints are required?
- Answer: Lock the ontology release version used for a run (see `ontology/releases/`). Record and surface the exact graph version, params file, and proto codegen commit in the replay metadata. Enforce backward-compatible graph edits; require migrations and golden-file updates for behaviour changes. Run `just gen-check` to ensure proto regeneration matches committed `proto/gen/`.
- Has solution: YES — ontology releases exist and deterministic testing practices are documented. Strict migration discipline must be followed.

2) Preventing string leakage into clock contract
- Question: The clock contract must contain no strings. How is this enforced automatically and how do you prevent accidental regression?
- Answer: Add a regression unit test asserting the `.proto` for the clock contains no string fields; this repo already references such a test in docs. Combine with `buf lint` in CI. If the test or lint is missing, add it to `just ci`.
- Has solution: PARTIAL — the design forbids strings and tests are referenced; verify CI contains the enforce rule.

3) Provenance enforcement across transformations
- Question: How do you prevent a component from promoting a PROJECTED statement to MEASURED or fusing different provenance classes into a single surfaced claim?
- Answer: Implement type-rich message envelopes carrying provenance class at the wire (proto messages with provenance enum) and have each transform assert and propagate provenance. At surfacing, format and label each statement with its provenance. Tests should reject any path that narrows provenance.
- Has solution: PARTIAL — provenance model is documented; review code paths to ensure all proto messages include provenance and tests block narrowing.

4) Scaling identity resolution at high cardinality
- Question: Identity joins can be N^2 if naive. How to scale joins for very large clusters with millions of entities and metrics?
- Answer: Use locality-aware indexing (namespace/node partitions), bloom filters for candidate pruning, and incremental join windows. Precompute stable identity keys (e.g., pod UID + node) and only fallback to label heuristics when keys differ. Shard join processing by topology to bound pairwise costs.
- Has solution: PARTIAL — identity package exists but large-scale optimizations may need attention and benchmarking.

5) Enforcing no learned thresholds policy
- Question: How is the policy "thresholds come from customer config, never from learned behaviour" enforced in code and review?
- Answer: Detection rules read threshold values exclusively from params/declared config; any learned suggestion code must write to an advisory surfaced channel (author-reviewed) rather than changing live thresholds. CI/code review should flag code that writes derived thresholds into detection rule state.
- Has solution: PARTIAL — policy is in docs; automated enforcement via code scanning/CI may need more work.

6) Hot reload of authored edges and bindings
- Question: Can the graph overlays or authored bindings be hot-reloaded without restarting `obsd`? What's the safe reapply strategy?
- Answer: Implement a controlled reload path: validate incoming overlay with graphlint, snapshot new graph release, run a dry-run detection in a separate process, then atomically swap the in-memory graph when validation passes. If not implemented, require rolling-restart with the new committed release.
- Has solution: NO / Needs attention — docs discuss releases but hot-reload automation likely needs engineering.

7) Auditability and explanation artefacts
- Question: For every finding, what artefacts should be stored for audit (inputs, graph version, selection decisions, fingerprint bytes)? How long are they retained and accessible?
- Answer: Store compact digests: timestamps, graph release id, selection decisions, fingerprint hash and the minimal raw windows. Retain per-customer storage according to retention policy; expose via API for backtest/replay. If retention or API missing, mark as needs attention.
- Has solution: PARTIAL — some metadata is recorded for replay; retention policy and API surfaces may need attention.
