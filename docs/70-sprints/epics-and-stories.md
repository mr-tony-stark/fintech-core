# Epics & Stories Status

This page summarizes high-level Epics and Stories derived from the Storo nucleus docs. It is the living status reference for planning and tracking at the Epic/Story level. Dependencies are listed at the end.

## Contracts-First (storo-specs)
- Description: Single source of truth for events, APIs, fixtures, and codegen used by all services.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Event envelope v1 finalized | Lock `events.md` envelope and versioning policy | `docs/20-specs/events.md` includes "v" convention; additive/breaking rules; examples; referenced by component docs |
| Canonical Transfer API defined | Confirm `POST/GET /transfers` schema and examples | `docs/20-specs/api-canonical-transfer.md` complete; example requests/responses validated |
| Golden fixtures catalog | Commit sample events/transfers for CI contract checks | Fixtures exist and are referenced by components/CI guidance in `ci-cd.md` |
| Codegen targets listed | Publish Go/TS targets and tag rules | `ADR-0003` next steps reflected; pinning/versioning in `release-strategy.md` |
| Contract validation in CI | Each service validates in/out payloads in CI | `ci-cd.md` shows `contract-validate` job usage |

## Platform/Base
- Description: Shared admin, time, errors, IDs, migrations used by all services.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Admin endpoints standard | `/live`, `/ready`, `/metrics`, `/version` contract | `docs/10-components/platform-base.md` details interface; components cite adoption |
| Banking time & calendars | ZA/ZW calendars and helpers shipped | Regions/timezones documented; Directory/CTS reference usage |
| Errors + IDs conventions | Error taxonomy and ID/tracing helpers | Usage in `platform-base.md`; linked to `20-specs/error-codes.md` |
| Embedded migrations guidance | Standard migrations story for services | Example present; referenced by components |

## Event Bus & Outbox
- Description: Exactly-once event delivery and idempotent consumption.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Outbox pattern mandated | ADR and infra page finalized | `ADR-0001` accepted; `10-components/event-bus-outbox.md` complete |
| Outbox table + dispatcher | Standard schema/dispatcher responsibilities | Data model + sequence documented; metrics + DLQ noted |
| Consumer idempotency rules | Dedupe by `eventId` and lifecycle uniqueness | Rules in `events.md`; components reference |
| Observability for bus | Backlog, publish lag, consumer lag metrics | Named metrics and alerts in `observability.md` |

## Canonical Transfer Service (CTS)
- Description: Unified API and lifecycle orchestrator.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| API endpoints | Define `POST /transfers`, `GET /transfers/:id` | `api-canonical-transfer.md` complete and linked |
| Idempotency rules | Enforce request dedupe and collision behavior | Behavior documented with test outline in `canonical-transfer-service.md` |
| Pre-screen compliance | Mandatory `/screen` before submit | Allow/deny flow and failure modes documented; sequence shows deny path |
| Directory routing | Deterministic routing via Directory | CTS↔Directory contract captured |
| Outbox events | Emit `transfers.*` via outbox | Envelope fields per `events.md`; outbox table documented |
| State machine | INITIATED→SUBMITTED→ACCEPTED→SETTLED/RETURNED/FAILED | Mermaid in `30-diagrams/lifecycle-state.mmd` referenced |

## Ledger Service
- Description: Double-entry, append-only book of record.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Posting rules mapped | Apply `posting-rules.md` on events | `ledger-service.md` links to `posting-rules.md`; examples included |
| Double-entry invariants | Balanced journals, append-only, idempotency | Invariants present; idempotency keying documented |
| Event consumption | Consume accepted/settled/returned | Contracts and dedupe rules referenced |
| Read APIs | Balances, journal, statements | Read endpoints defined |
| Observability | Posting latency, drop rate metrics | Metrics listed; SLOs set in `ledger-service.md` |

## Compliance Screening
- Description: Screen payer/payee against sanctions lists with local index.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| List ingest & index | Sources, normalization, versioning | Sources and index documented; SLAs noted |
| /screen API | Low-latency allow/deny responses | API contract and response fields present |
| Delta re-screen | Re-screen on list updates | Events and operator flow documented |
| Observability & failures | Stale index alerts, match rate | Metrics and failure modes covered |

## Directory & Routing
- Description: Authoritative institution/BIN/fees/windows and routing.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Routing API | Define `/routes` lookup | Request/response structure documented |
| Versioning & refresh | Data versions, checksums, updates | `directory.version.updated` event documented |
| Calendars | Publish ZA/ZW calendars | Calendar API documented |
| Data model | Institutions/BINs/fees/windows | Schema and audit/versioning captured |

## Reconciliation & Returns
- Description: Ingest statements, match transfers, raise settlement/return events.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Statement ingest | Normalize per-rail files/APIs | Per-rail specs referenced |
| Matching engine | Composite keys and timelines | Matching criteria and flows documented |
| Exception queue | Operator queue for unmatched | Event and operator flow present |
| Return events | Emit `transfers.returned` | Ledger reversal linkage described |

## Rail Gateways (Foundation)
- Description: Shared gateway template and quality bars.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Gateway template | Standard structure, outbox, observability | Referenced across `rail-gateway-*.md` |
| Validation discipline | Strict schema and idempotency | Transform rules present |
| Webhooks & signatures | Verify, persist redacted artifacts | Webhook contracts + security documented |
| Reason code mapping | Partner → Storo mappings | `error-codes.md` referenced per gateway |

## Rail Gateway — USDC/Algorand
- Description: On-chain USDC transfers and confirmations.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Consume submitted.usdc | Validate and build ASA txn | Inputs/validation rules documented |
| Submit & confirm | Submit, confirm N blocks | `accepted/settled/failed` flows documented |
| Data model | Tx metadata, status | Tables defined; artifacts encrypted |
| Security | Keys/signing isolation | Security practices documented |

## Rail Gateway — EcoCash
- Description: STK/USSD prompt and callbacks.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| STK submit | Handle prompt initiation | Sequence diagram present |
| Callback handling | Map statuses to events | Accepted/settled/returned mapping documented |
| Data model | Ops table, refs, reason codes | Tables and retention noted |

## Rail Gateway — OPPWA
- Description: Card/tokenized flows with auth/capture and returns.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Auth/Capture mapping | Intent → OPPWA operations | Mapping table and flows present |
| Webhooks | Verify and correlate | Webhook contract and security documented |
| Reason mapping | Result/return → Storo reason | Versioned mapping table referenced |
| PCI containment | Scope kept to gateway | PCI guidance aligns in security doc |

## Rail Gateway — Zimswitch
- Description: ISO discipline, settlement files, returns.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| ISO field rules | Validate ISO-like fields | Field mapping documented |
| Settlement ingest | Daily settlement and returns | Recon integration points documented |

## Rail Gateway — EFT (Bankserv)
- Description: Batch submissions, settlement/returns, DebiCheck.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Batch submit | Produce/validate files | File layout and checksum rules documented |
| Returns ingest | Map returns to events | Reason mapping and timelines documented |
| DebiCheck support | Mandate lifecycle | Mandate rules documented |

## Rail Gateway — PayShap
- Description: Proxy resolution and instant payment.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Proxy resolution | Resolve proxy to account | Directory integration documented |
| Submit/settle | Events and timeouts | Sequence present; failure modes documented |

## Rail Gateway — MTN MoMo & Airtel Money
- Description: Mobile money gateway parity.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Submit & callbacks | Submit PUSH; process callbacks | Event mapping documented |
| Data model & idempotency | Ops table; partner refs | Tables and idempotency rules present |

## Regulatory Reporting
- Description: BoP and goAML reporting pipelines and ops.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| BoP pipeline | Build and submit BoP payloads | Fields/cadence in `20-specs/regulatory-reporting.md` |
| goAML STR/CTR | Thresholds and submission | Ops runbook plus spec present |
| Receipts & DLQs | Immutable artifacts and retries | Data model and runbooks documented |

## Observability
- Description: Metrics, logs, tracing, dashboards, alerting.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Golden signals | Define per component | `40-ops/observability.md` lists metrics and alerts |
| Dashboards | Exec/SRE/Finance dashboards | Dashboard sections enumerated |
| Tracing | Propagate trace context | Tracing conventions documented |

## Security & Privacy
- Description: Secrets, mTLS, RBAC, PCI scope, POPIA, retention.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Secrets management | Vault/SSM patterns | `40-ops/security.md` complete |
| Service authN/Z | mTLS or mesh JWT | Enforcement points documented |
| PCI scope | Card data isolation in gateways | PCI guidance present; logs redaction noted |
| Data retention & PII | Retention windows, redaction jobs | `20-specs/data-retention-pii.md` complete; DSAR runbook added |

## Risk Limits & KYC Tiers
- Description: Tiered onboarding and velocity controls.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Tier matrix | T0/T1/T2 rules & limits | `20-specs/risk-limits.md` complete |
| Enforcement points | CTS and Compliance gates | Enforcement hooks documented |

## FX Policy
- Description: Rate sources, TTL, audit, controls.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Quote sources | RBZ vs market providers | `20-specs/fx-policy.md` lists sources/provenance |
| TTL & audit | TTLs and event tagging | Events reference optional FX fields where relevant |

## Multi-Repo & CI/CD
- Description: Independent repos, CI pipelines, releases.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Repo map & ownership | Clear list and owners | `50-repo-structure/multi-repo-architecture.md` complete |
| CI standard | Lint/test/contract/build/publish | `50-repo-structure/ci-cd.md` finalized; example YAML included |
| Release strategy | Semver, dual-publish, deprecation | `50-repo-structure/release-strategy.md` complete |

## Local Dev (storo-devstack)
- Description: Run the fleet locally and swap one service.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| Compose stack | Postgres, LocalStack, Prom, Grafana | `50-repo-structure/local-dev.md` documents bring-up |
| Swap one service | Disable container, run local | Override example documented |
| Smoke test | POST transfer through USDC slice | Steps and expected events documented |

## Documentation & Diagrams
- Description: Keep docs executable and coherent.

| Story | Description | Acceptance Criteria |
|-------|-------------|---------------------|
| ADRs in place | ADR-0001/2/3 committed | ADRs present and referenced |
| Component pages | All `10-components/*.md` complete | Purpose, interfaces, data, failure modes sections complete |
| Diagrams updated | Component, lifecycle, sequences | Mermaid sources in `30-diagrams/` referenced |
| Glossary/NFRs | Shared language and targets | `glossary.md`, `nfrs.md` up to date |

## Cross‑Epic Dependencies
- Contracts-First → prerequisite for CTS, Ledger, Gateways, Recon, Regulatory
- Platform/Base → adopted by all services before health/metrics are relied on
- Event Bus & Outbox → required before reliable domain events
- Directory & Compliance → required before CTS can submit to gateways
- Posting Rules (spec) → required before Ledger can consume events
- Rail Gateways → depend on CTS events and Directory; Recon depends on gateways’ files/events
- Regulatory Reporting → depends on events and Ledger read APIs
- Operator Console → depends on CTS/Ledger/Recon APIs and events
- Observability/Security → cross-cutting; adoption required before go-live
