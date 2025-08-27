# Rail Gateway — MTN MoMo (Mobile Money)

**Purpose**  
Translate canonical transfers to MTN MoMo APIs (P2P/P2M), enforce validation, process webhooks, and emit events.

## Responsibilities
- Consume `transfers.submitted.mtnmomo`.
- Initiate payment request (PUSH), handle consent/approval callbacks.
- Emit lifecycle events; persist artifacts.

## Interfaces
### Inputs
- Events: `transfers.submitted.mtnmomo`
- HTTP: `POST /webhooks/mtnmomo`

### Outputs
- Events: `transfers.accepted`, `transfers.settled`, `transfers.returned`, `transfers.failed`

## Data Model
- `mtnmomo_ops` (...)

## Rules
- MSISDN format validation; currency allowlist.
- Idempotency via partner reference.

## Failure Modes & Retries
- 202/accepted without finalization → poll until terminal.

## Observability & Security
- Metrics, DLQ; webhook signature verify, redaction.
