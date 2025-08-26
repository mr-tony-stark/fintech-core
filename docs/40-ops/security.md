# Security

Standards for secrets, access, PCI scope, and PII handling in Storo.

---

## Secrets Management

- Stored in a centralized vault (e.g., HashiCorp Vault).  
- Rotated at least every 90 days; immediate rotation on incident.  
- No secrets in environment variables for long-lived services without vault agent.  

---

## Access Control

- Service-to-service auth via mTLS or mesh-issued JWT.  
- RBAC for Operator Console (OPS, COMPLIANCE, ADMIN).  
- Principle of least privilege to DBs and blob stores.

---

## PCI & PAN Scope

- Card data handled exclusively in **OPPWA/Zimswitch gateway**.  
- PAN/token never flows into CTS or Ledger.  
- Webhook/file artifacts are redacted and encrypted.

---

## PII Handling

- Encrypt at rest; redact in logs and events.  
- Minimize in DBs; store raw artifacts in encrypted blob store.  
- Data retention per `20-specs/data-retention-pii.md`.

---

## Secure Coding

- Input validation at boundaries; strict schema checks.  
- Dependency scanning and SAST/DAST in CI.  
- Security headers on all admin endpoints.

---

## Incident Response

- 24/7 on-call rotation; breach protocol documented.  
- Forensics: immutable logs and event store.  
- Post-incident ADR if architectural changes required.
