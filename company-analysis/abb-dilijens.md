ABB + Dilijens — Diligence & Integration Checklist

Purpose: operational checklist for our team (`Dilijens`) when engaging ABB — PoC plan, integration steps, stakeholder asks, risks, and acceptance criteria.

1) PoC scope and success criteria
- Define hypothesis, measurable success metrics (precision, latency, energy reduction), duration (typical 4–12 weeks), required data feeds, and executive sponsor.

2) Minimal technical artifacts to obtain
- Telemetry samples, API/Protobuf schemas, device/firmware upgrade process, network diagrams, security posture, and staging/test environment access.

3) Data access & security constraints
- Plan for outbound-only ingestion if OT networks are constrained. Verify encryption (TLS), mutual auth, and any air-gap requirements. Request data retention and ownership terms.

4) Integration touchpoints & adapters
- Ingest endpoints (MQTT, gRPC/Protobuf), device management APIs (REST), and potential protocol translators for Modbus/OPC UA. Design edge aggregation to reduce telemetry volume.

5) Runbook & operational readiness
- Define runbooks for onboarding, rollback, and incident response. Identify internal owners (sales lead, solutions architect, legal) and ABB counterparts (product owner, integration architect).

6) Risk mitigations
- Use replayed or mirrored test data; start read-only; isolate with a data diode or outbound-only path; schedule PoC in a maintenance window; define rollback criteria.

7) Commercial and legal red flags
- Watch for restrictive data-ownership clauses, long lock-in, excessive indemnities, export-control constraints, and unclear uptime SLAs.

8) Acceptance criteria for pilot → production
- Clear thresholds for detection performance, validated upgrade/rollback, documented runbooks, monitoring and alerting in place, security review passed, and a signed support contract.

9) Timeline & checkpoints
- PoC: 4–12 weeks. Pilot: 3–6 months. Production ramp: depends on procurement (3–12 months). Set review gates at PoC weeks 2, 6, and at pilot kickoff.

10) Interview checklist for ABB teams
- Ask for API contracts, sample telemetry, observability story, test harnesses, firmware-update cadence, and a named SME (Subject Matter Expert) for escalation.

11) Deliverables from Dilijens
- PoC plan & runbook, test harness for replay, monitoring dashboards, alerting rules, security assessment, and handover documentation for ops.

12) Quick red-flag filter for leadership
- If ABB cannot provide sample telemetry, refuses a staging environment, or imposes restrictive data clauses, flag for commercial review.

13) Next actions template
- Request artifacts, schedule a 60-minute technical kickoff (include product owner + integration architect + Dilijens solutions architect), agree PoC timeline, and assign internal owners.
