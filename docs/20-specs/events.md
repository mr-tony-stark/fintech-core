# Event Specifications

The **Event model** defines the envelope and catalog of all domain events emitted across Storo services.

---

## 🎯 Purpose
- Provide a **single canonical envelope** for all events.  
- Define the **catalog of event types** (`transfers.*`, `ledger.*`, etc.).  
- Ensure **idempotency and consistency** across services.  

---

## 📦 Event Envelope (v field)

```json
{
  "eventId": "uuid",
  "type": "transfers.settled",
  "v": 1,
  "occurredAt": "2025-08-26T10:15:01Z",
  "transferId": "tr_123",
  "tenantId": "tn_456",
  "payload": { "amount": { "value": 1000, "currency": "USD" } }
}
```

**Fields**
- `eventId` – unique ID (UUIDv7 recommended).  
- `type` – dot-delimited string category (see catalog).  
- `v` – envelope schema version (integer, starting at 1).  
- `occurredAt` – ISO8601 UTC timestamp.  
- `transferId` – optional link to transfer (if relevant).  
- `tenantId` – tenant scoping.  
- `payload` – type-specific content.  

**Optional enrichment (additive)**
- `kycTier` – KYC tier for context (T0|T1|T2).  
- `riskScore` – screening risk score (0-100).  
- `exchangeControlRef` – reference for exchange control/BoP.  
- `taxCode` – tax/VAT code for fee lines.  
- `proxyType` – proxy type for PayShap (cell|email|id).  

> Additive fields do not break consumers; treat unknown fields as optional.

---

## 📚 Event Catalog

### Transfers
- `transfers.initiated`  
- `transfers.submitted.<rail>`  
- `transfers.accepted`  
- `transfers.settled`  
- `transfers.returned`  
- `transfers.failed`  

### Ledger
- `ledger.posting.created`  
- `ledger.balance.updated`  

### Compliance
- `compliance.entity.flagged`  

### Reconciliation
- `recon.statement.ingested`  
- `recon.exception.opened`  

### Directory
- `directory.version.updated`  

---

## 🔁 Idempotency Rules
- Event consumers must dedupe using `eventId`.  
- For transfer lifecycle, `(transferId,type)` must be unique.  
- Outbox pattern ensures atomic persistence + publish.

---

## 🧭 Versioning
- Envelope: `v` is the envelope schema version (current: 1).  
- Additive changes (new optional fields) require no version bump.  
- Breaking payload changes introduce a new `type` version (e.g., `transfers.settled.v2`) with dual-publish during migration.

---

## 📊 Observability
- Metrics: events/sec by type, lag from occurredAt → consumed.  
- Audit: immutable event store recommended for replay/debug.

---
