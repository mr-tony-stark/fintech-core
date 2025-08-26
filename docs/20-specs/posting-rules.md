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
# Posting Rules (Storo)

Mapping of domain events → ledger postings.  
Ensures double-entry and accrual accounting discipline.

---

## Transfers

**Event: transfers.accepted (AUTH)**  
- Create memo postings (off-balance).  
- No balance impact until SETTLED.

**Event: transfers.settled (PUSH)**  
- Debit: Payer (User Balance)  
- Credit: Merchant Payable (Liability)  
- Debit: Merchant Payable  
- Credit: Liquidity (Asset)  

**Event: transfers.settled (PULL)**  
- Debit: Liquidity (Asset)  
- Credit: User Balance (Liability)

**Event: transfers.returned**  
- Reverse prior postings with contra entries.  
- Debit/Credit Chargebacks Payable as needed.  

---

## Fees

**Transaction Fee**  
- Debit: User Balance (Liability)  
- Credit: Transaction Fees Earned (Revenue)

**FX Spread**  
- Debit: Liquidity (Asset)  
- Credit: FX Spread Income (Revenue)

---

## Reconciliation Adjustments

**Unmatched line item (temporary)**  
- Debit: Suspense (Asset)  
- Credit: Suspense Offset (Liability)  
- Must be cleared before books close.  

---

## Closing Entries

**Close Revenue**  
- Debit: Transaction Fees Earned  
- Credit: Net Income  

**Close Expenses**  
- Debit: Net Income  
- Credit: Processing Costs / Chargebacks Losses  

**Close Net Income**  
- Debit: Net Income  
- Credit: Retained Earnings  

---

## Notes
- Posting rules are immutable and versioned.  
- Each posting must include: `transferId`, `eventId`, `occurredAt`, memo.  

