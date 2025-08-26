# Posting Rules

Posting rules define how domain events are transformed into double-entry ledger postings.

---

## 🎯 Purpose
- Ensure **deterministic accounting** for every event.  
- Encode **business logic** (fees, FX, reversals).  
- Provide a testable mapping for CI.

---

## 🔀 Event → Posting Map

### `transfers.settled`
Debit: Payer  
Credit: Merchant  
Credit: Fees (if applicable)  

### `transfers.returned`
Reverse prior settlement postings (contra entries).  

### `transfers.accepted` (auth rails only)
Debit: UserAuthHold (memo)  
Credit: LiquidityPending (memo)  

---

## 💱 Multi-Currency Rules
- Settlement uses **fxQuote** at time of event.  
- Residual differences → `FX_PNL`.  
- Rounding differences → `FX_PNL_ROUNDING`.  

---

## 📐 Example Table

| Event                | Debit          | Credit        | Amount  | Notes         |
|----------------------|----------------|---------------|---------|---------------|
| transfers.settled    | USER           | MERCHANT      | 10000   | Net to merchant |
|                      |                | FEES          | 100     | Fee captured |
| transfers.returned   | MERCHANT       | USER          | 10000   | Contra |
|                      | FEES           | USER          | 100     | Fee reversal |

---

## 🧪 Test Fixtures
- Deterministic fixtures in `/testdata/ledger/`.  
- CI asserts byte-for-byte journal correctness.

---
