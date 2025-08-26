# Data Retention & PII Handling

The **data retention policy** governs how long Storo stores sensitive data and how it is redacted.

---

## 🎯 Principles
- **Minimize PII** stored in core DBs.  
- **Encrypt at rest** all raw rail payloads.  
- **Retain only what’s needed** for audit, compliance, dispute resolution.  
- **Expire or redact** data after retention window.

---

## 📦 Storage Classes
- **Canonical DBs (CTS, Ledger, Compliance, Directory, Recon)**  
  - Store IDs, references, metadata only.  
  - No raw PII beyond accountId/tenantId.  

- **Blob Store (encrypted)**  
  - Stores raw rail payloads, statements.  
  - Access tightly controlled, time-limited.  

---

## ⏳ Retention Windows
- Transfer & ledger events: **7 years** (audit requirement).  
- Raw rail payloads: **18 months**.  
- Compliance screening results: **5 years**.  
- Operator actions (audit log): **7 years**.  

---

## 🧹 Redaction
- After expiry, replace PII fields with irreversible hashes.  
- Keep metadata (transferId, amounts, dates).  

---

## 🔐 Security
- All PII encrypted at rest + in transit.  
- Logs redact sensitive fields (names, IDs, PANs).  
- Access scoped to tenant & role.  

---

## 🧭 Runbooks
- **Retention job failure** → retry, escalate if backlog > 24h.  
- **Legal hold** → suspend deletion for specific entities.  
- **PII exposure incident** → trigger breach protocol immediately.

---
