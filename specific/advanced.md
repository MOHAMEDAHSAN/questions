# Advanced Specific (Architecture / Design) Questions & Answers

Advanced architecture and design prompts with model answers and a clear "Has solution" status. Use these for deep technical interviews and design reviews.

---

1) Deterministic replay under graph evolution

Question: How do you guarantee byte-identical deterministic outputs when the ontology graph changes between runs? What versioning and compatibility constraints are required?

Answer: Lock the ontology release used for a run (see `ontology/releases/`). Record and surface the exact graph version, params file, and proto codegen commit in replay metadata. Enforce backward-compatible edits, require migrations and golden-file updates for behavior changes, and run `just gen-check` to ensure proto regeneration matches committed `proto/gen/`.

 
---

2) Preventing string leakage into clock contract

Question: The clock contract must contain no string fields. How do you enforce this automatically and prevent regressions?

Answer: Add a regression test that parses compiled proto descriptors and asserts no fields are typed as `string`. Run `buf lint` in CI to catch schema regressions.

 
---

3) Provenance enforcement across transformations

Question: How do you prevent a component from promoting a PROJECTED statement to MEASURED or fusing different provenance classes into one surfaced claim?

Answer: Use typed proto envelopes carrying provenance enums and require transforms to assert/propagate provenance. Surfacing formats must label provenance and tests should reject any path that narrows provenance.

 
---

4) Scaling identity resolution at high cardinality

Question: Identity joins can be N^2 if naive. How do you scale joins for very large clusters with millions of entities and metrics?

Answer: Use locality-aware indexing (namespace/node partitions), bloom filters for candidate pruning, incremental join windows, precomputed stable identity keys (pod UID + node), and shard join processing by topology to bound pairwise costs.

 
---

5) Enforcing no learned thresholds policy

Question: How is the policy "thresholds come from customer config, never from learned behaviour" enforced in code and review?

Answer: Detection reads thresholds exclusively from declared params. Learned suggestions must appear only in advisory channels (AUTHORED) and require human review before being applied. CI should flag code that writes derived thresholds into live detection rules.

 
---

6) Hot reload of authored edges and bindings

Question: Can graph overlays or authored bindings be hot-reloaded without restarting `obsd`? What is a safe reapply strategy?

Answer: Implement controlled reload: validate overlays with graphlint, snapshot a new release, run dry-run detection in an isolated process, and atomically swap the in-memory graph on success. Otherwise require a rolling restart with the new committed release.

 
---

7) Auditability and explanation artefacts

Question: For every finding, what artefacts should be stored for audit (inputs, graph version, selection decisions, fingerprint bytes)? How long are they retained and accessible?

Answer: Store compact digests: timestamps, graph release id, selection decisions, fingerprint hash, and minimal raw windows. Retain per-customer per policy and expose via API for backtest/replay; add audit retention policy documentation.

 
