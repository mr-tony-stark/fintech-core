# Platform/Base Library

The **Platform/Base Library** provides shared utilities for all Storo services, ensuring consistency in health checks, IDs, error handling, time logic, and migrations.

---

## 🎯 Purpose
- Provide common **admin endpoints** for all services.  
- Standardize **time, IDs, errors** across the stack.  
- Support embedded **migrations/configs**.  
- Simplify **observability** and operational tooling.

---

## 🛠 Responsibilities
- Expose `GET /live`, `GET /ready`, `GET /metrics`, `GET /version`.  
- Provide banking-aware time utilities (cutoffs, holidays).  
- Standardize error handling (ErrorList, ParseError).  
- Offer ID generation and request tracing.  
- Embed DB migrations in service binaries.  

> Ship canonical holiday calendars for ZA/ZW and banking-day helpers (`Africa/Johannesburg`, `Africa/Harare`).

---

## 🔌 Interfaces
- Imported as library into all services.  
- Provides helpers for admin, time, error, IDs, migrations.  

---

## 📐 Example

```go
// Health server
admin := base.NewAdminServer(":8081")
admin.AddLivenessCheck("db", db.Ping)
admin.Start()

// Time utils
t := base.Now()
if t.IsBankingDay() { ... }
```

---

## 🗄 Modules
- **Admin** → liveness, readiness, metrics, version.  
- **Time** → banking days, holidays, cutoffs.  
- **Errors** → ErrorList, ParseError.  
- **IDs** → ID(), correlation IDs.  
- **Migrations** → embedded FS for DB schema upgrades.  

---

## 📊 Observability
- Uniform `/metrics` for Prometheus.  
- Structured error/logging helpers.  
- Request tracing correlation.  

---

## 🔐 Security
- Admin endpoints on localhost or service mesh only.  
- No sensitive data in metrics/logs.  

---

## 🧭 Runbooks
- **/live fails** → service not running; check logs.  
- **Migration failed** → rollback DB version, re-run migration.  
- **Clock drift** → verify NTP sync; banking-day logic depends on accurate time.

---
