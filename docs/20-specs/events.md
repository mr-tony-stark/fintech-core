# Event Specifications

The **Event model** defines the envelope and catalog of all domain events emitted across Storo services.

---

## 🎯 Purpose
- Provide a **single canonical envelope** for all events.  
- Define the **catalog of event types** (`transfers.*`, `ledger.*`, etc.).  
- Ensure **idempotency and consistency** across services.  

---

## 📦 Event Envelope

```json
{
  "eventId": "uuid",
  "type": "transfers.settled",
  "occurredAt": "2025-08-26T10:15:01Z",
  "transferId": "tr_123",
  "tenantId": "tn_456",
  "payload": { ... rail or business-specific data ... }
}
```

**Fields**
- `eventId` – unique ID (UUIDv7 recommended).  
- `type` – dot-delimited string category (see catalog).  
- `occurredAt` – ISO8601 UTC timestamp.  
- `transferId` – optional link to transfer (if relevant).  
- `tenantId` – tenant scoping.  
- `payload` – type-specific content.

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

## 📊 Observability
- Metrics: events/sec by type, lag from occurredAt → consumed.  
- Audit: immutable event store recommended for replay/debug.

---
