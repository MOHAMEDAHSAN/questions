# Specific / Architecture and Design Questions


1. Tech choices
- Question: Why did you choose Go for `obsd` and Python for `clockd`? Trade-offs in performance, ecosystem, and deployment?
- Answer: Go gives a small, CGO-free single binary suitable for deterministic, high-concurrency runtime and easy cross-compilation for `obsd`. Python fits rapid experimentation and rich ML libraries for `clockd`. Trade-offs: Python adds process isolation and dependency management, Go requires stricter typing but yields robust production binaries.
- Has solution: YES — current split matches repo layout and deployment expectations.

- Question: Why use Protobuf/buf for contracts instead of JSON/REST? How does committed codegen (`proto/gen/`) help cross-language determinism?
- Answer: Protobuf ensures a strict schema, language-native types, and deterministic serialization. Committed `proto/gen/` prevents drift between languages and CI `buf` checks guard contract changes.
- Has solution: YES — proto/buf and committed codegen are in place.

2. Determinism and guarantees
- Question: Explain the determinism guarantee: what inputs must match to obtain identical outputs in `replay` mode?
- Answer: Same readings, same graph release, same params, same topology snapshot, and identical proto codegen produce identical fingerprints and findings. Clock presence must be controlled (injected or absent) for parity.
- Has solution: YES — replay philosophy and golden fixtures are central.

- Question: How do you ensure no string fields leak into the clock contract (proto/vigil/clock/v1/clock.proto)? How is this enforced in tests?
- Answer: Add a unit test that parses the compiled proto descriptors and asserts no `string` typed fields exist; run `buf lint` as part of CI. 
- Has solution: PARTIAL — design forbids strings and docs reference a conformance test; add explicit automated test if absent.

3. Data modeling and provenance
- Question: Describe how provenance classes (MEASURED / PROJECTED / AUTHORED) are assigned and enforced. Give examples of what transformations are allowed per class.
- Answer: Assign provenance at ingestion and carry it through proto envelopes. MEASURED: raw metrics or deterministic arithmetic; PROJECTED: clock outputs with uncertainty; AUTHORED: manual graph edges or annotations. Enforcement: transforms must assert provenance—no narrowing allowed.
- Has solution: PARTIAL — provenance model is documented; audit code paths to ensure envelopes are used everywhere.

- Question: How do you prevent fusing statements from multiple provenance classes into a single claim at surfacing time?
- Answer: Surfacing layer must present grouped claims by provenance and reject any merger; UI and API formats should label each claim and keep them adjacent but distinct.
- Has solution: PARTIAL — renderer files exist; add surface-level tests to prevent accidental fusion.


4. Thresholds and configuration
- Question: Where do thresholds come from (customer config vs learned)? What happens when nothing is declared?
- Answer: Thresholds come from declared customer config first. If none declared, surface as unbounded and ineligible for certain gated detections. 
- Has solution: YES/PARTIAL — documented policy exists; ensure UI shows unbounded status.

- Question: How does the system prevent learned thresholds from leaking into the detection rules?
- Answer: Learnings may only appear in advisory channels (AUTHORED suggestions) and never be written into live detection params without human review and commit.
- Has solution: PARTIAL — policy exists; implement code/CI checks to prevent accidental writes.

5. Integration and APIs
- Question: What surface APIs and outputs are available to downstream systems (alerts, web surfaces)? How are bindings and finds rendered (see `cmd/obsd` rendering files)?
- Answer: `obsd` provides renders (inventory, findings, selection) and can output via API/CLI. Bindings are rendered per the renderer templates and feed downstream alerts. Document API endpoints and example payloads for integrators.
- Has solution: PARTIAL — renderers exist; API docs and example payloads may need expansion.

- Question: How would you version and roll out a change to the graph schema (`ontology/graph`)?
- Answer: Create a new ontology release, run lint and harness tests, update `ontology/releases` with migration notes, regenerate `proto/gen/` if needed, and roll out with canary validation via replay tests.
- Has solution: YES — releases pattern exists; formal migration checklist should be followed.

6. Testing and CI
- Question: What are the key unit/integration tests required to maintain the determinism guarantee? How do you use `testdata/` golden fixtures?
- Answer: Unit tests for selection, identity, fingerprint; integration replay tests comparing outputs to golden fixtures. Golden fixtures are the canonical outputs checked into `testdata/` and used by `just ci`.
- Has solution: YES — harness and golden fixtures exist; keep them updated with schema changes.

- Question: How do you run `just ci` locally and what does it validate?
- Answer: Run `just ci` to run lint, gen-check, and `go test -race ./...`. It validates proto codegen parity, formatting, and deterministic test suites.
- Has solution: YES — `just` recipes are present; verify environment prerequisites (Go, buf, etc.).
