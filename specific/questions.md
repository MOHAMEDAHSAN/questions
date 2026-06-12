# Specific / Architecture and Design Questions
Clear architecture and design prompts with concise answers and a support status. Use these for deep-dive interviews or design reviews.

---

1) Tech choices
Question: Why use Protocol Buffers (Protobuf) / `buf` for contracts instead of JSON/REST? How does committed codegen (`proto/gen/`) help cross-language determinism?

Answer: Protocol Buffers (Protobuf) enforce a strict schema, language-native types, and deterministic serialization. Committed `proto/gen/` prevents drift between languages and CI (Continuous Integration) `buf` checks guard contract changes.


Question: Why did you choose Go for `obsd` and Python for `clockd`? Trade-offs in performance, ecosystem, and deployment?

Answer: Go gives a small, CGO-free single binary suited for deterministic, high-concurrency runtime and easy cross-compilation for `obsd`. Python enables rapid experimentation and access to ML libraries for `clockd`. Trade-offs: Python adds dependency management and a separate process; Go gives stronger build-time guarantees and a simpler production footprint.

 
---



---

2) Determinism and guarantees

Question: Explain the determinism guarantee: what inputs must match to obtain identical outputs in `replay` mode?

Answer: Identical readings, the same ontology release, the same params, the same topology snapshot, and identical proto codegen produce identical fingerprints and findings. The clock must be injected or absent consistently for parity.

 
---

Question: How do you ensure no string fields leak into the clock contract (proto/vigil/clock/v1/clock.proto)? How is this enforced in tests?

Answer: Add a unit test that parses compiled proto descriptors and asserts no `string` typed fields exist. Run `buf lint` in CI to catch regressions.

 
---

3) Data modeling and provenance

Question: Describe how provenance classes (MEASURED / PROJECTED / AUTHORED) are assigned and enforced. Give examples of what transformations are allowed per class.

Answer: Assign provenance at ingestion and carry it in proto envelopes. MEASURED are raw facts or deterministic arithmetic; PROJECTED are clock outputs with uncertainty; AUTHORED are human-created graph edges/notes. Transforms must propagate provenance and never narrow it.

 
---

Question: How do you prevent fusing statements from multiple provenance classes into a single claim at surfacing time?

Answer: Surfacing must present grouped claims by provenance and never merge them. UI (user interface) and API (Application Programming Interface) formats should label and keep claims adjacent but distinct.

 
---

4) Thresholds and configuration

Question: Where do thresholds come from (customer config vs learned)? What happens when nothing is declared?

Answer: Thresholds come first from declared customer config. When none is declared, surface the variable as "unbounded" and exclude from gated detections where appropriate.

 
---

Question: How does the system prevent learned thresholds from leaking into the detection rules?

Answer: Learned suggestions must surface in advisory channels (AUTHORED) and require human review before being applied to detection params. CI/code review should flag any code writing derived thresholds into live rules.

 
---

5) Integration and APIs

Question: What surface APIs and outputs are available to downstream systems (alerts, web surfaces)? How are bindings and findings rendered?

Answer: `obsd` provides rendered artifacts (inventory, findings, selection) and can output via API/CLI. Bindings and findings are rendered via templates in `cmd/obsd`; document endpoints and example payloads for integrators.

 
---

Question: How would you version and roll out a change to the graph schema (`ontology/graph`)?

Answer: Produce a new ontology release, run lint and harness tests, update `ontology/releases` with migration notes, regenerate `proto/gen/` if required, and canary via replay tests before full rollout.

 
---

6) Testing and CI

Question: What are the key unit/integration tests required to maintain the determinism guarantee? How do you use `testdata/` golden fixtures?

Answer: Unit tests for selection, identity, fingerprint; integration replay tests compare outputs to golden fixtures stored in `testdata/`. Keep fixtures updated and gate changes via review.

 
---

Question: How do you run `just ci` locally and what does it validate?

Answer: Run `just ci` to perform linting, `buf` gen-check, and `go test -race ./...`. It validates codegen parity, formatting, and deterministic tests.

 
