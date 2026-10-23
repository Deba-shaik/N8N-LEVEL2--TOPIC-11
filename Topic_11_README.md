# Topic 11 - Database Design for Automation

## README

**Platform:** n8n  
**Implementation:** In-memory workflow simulation using Code and IF nodes  
**Evidence:** `Topic_11_Evidence_Report.pdf`  
**Date tested:** 23 October 2026

## 1. Objective

Demonstrate relational-style order processing with a customer record, order header, two order items, credit-limit checking, conditional success/rejection routing, consolidated order output, and an audit trail.

> **Scope note:** This version does not connect to SQLite, PostgreSQL, or MySQL. It simulates relational data and transaction decisions in n8n. `COMMITTED` and `ROLLED_BACK` are output labels; no real SQL transaction is performed.

## 2. Workflow architecture

```text
Manual Trigger
  -> Code_InitRelationalDB
  -> Code_StageIncomingOrder
  -> Code_StageIncomingOrder1 (order header preparation; as labeled in screenshot)
  -> Code_InsertOrderItems
  -> Code_UpdateCustomerBalance
  -> IF_TransactionSucceeded
       TRUE  -> Code_QueryFullOrderJoin -> Code_FinalTransactionAudit
       FALSE -> Log_RollbackIncident
```

The workflow screenshot includes `Code_StageIncomingOrder1`; the screenshot, not the planned node name, is used here.

## 3. Node descriptions

| Node | Purpose |
|---|---|
| Manual Trigger | Start a test execution |
| Code_InitRelationalDB | Initialize sample customer/order structures for simulation |
| Code_StageIncomingOrder | Prepare customer ID, order ID, date, and two line items |
| Code_StageIncomingOrder1 | Prepare order header and calculate total (as configured in this workflow) |
| Code_InsertOrderItems | Build order-item records and validate total |
| Code_UpdateCustomerBalance | Calculate proposed balance and check credit limit |
| IF_TransactionSucceeded | Route eligible and rejected orders |
| Code_QueryFullOrderJoin | Combine staged order, customer, and item fields into one output; not a SQL JOIN |
| Code_FinalTransactionAudit | Check item total, credit limit, and success status |
| Log_RollbackIncident | Log credit-limit rejection details; not an actual database rollback |

## 4. Sample order data

- Order: `ORD-DB-5001`
- Customer for successful test: `CUST-001` (Acme Corp)
- Item 1: `SKU-SRV-01`, quantity 2, unit price 450, line total 900
- Item 2: `SKU-LIC-02`, quantity 5, unit price 75, line total 375
- Total: **1275**

## 5. How to test

1. Open the n8n workflow.
2. In the order staging node, set `customerId` to `CUST-001`.
3. Run **Execute workflow**. Check the True branch and `Code_FinalTransactionAudit`. Expected: `TRANSACTION_VERIFIED`, `totalMatches: true`, `creditValid: true`, `transactionValid: true`, total 1275, updated balance 2475.
4. Change the staged customer ID to `CUST-002`.
5. Run the entire workflow again. Check the False branch and `Log_RollbackIncident`. Expected: `ROLLED_BACK`, reason `Credit limit exceeded`, balance 450, limit 500.
6. Restore `CUST-001` for the default demonstration.

## 6. Test results (supported by screenshots)

| ID | Scenario | Actual result | Status |
|---|---|---|---|
| T01 | Eligible order / CUST-001 | `TRANSACTION_VERIFIED`; reported `COMMITTED` | PASS |
| T02 | Credit limit exceeded / CUST-002 | `ROLLED_BACK`; rejection logged | PASS |
| T03 | Line item sum | 900 + 375 = 1275, matching order total | PASS |
| T04 | Branching | Successful and rejected paths each observed | PASS |

## 7. Evidence files

`Topic_11_Evidence_Report.pdf` contains three provided screenshots: complete workflow, successful final audit, and rollback incident output. Each image has an explanation and a test matrix.

## 8. Limitations and next steps

- No live SQL database, SQL schema, persistent rows, enforced foreign keys, or SQL JOIN.
- No real ACID commit or rollback; output statuses reflect simulated decisions.
- Customer balances are calculated in the workflow; do not treat them as proof of durable database updates.
- For a database-backed implementation, configure database credentials and native database nodes, create `customers`, `orders`, and `order_items` tables with foreign keys, and execute real transactions with explicit commit/rollback.
- Export the workflow JSON directly from n8n and include it with the submission if required.

## 9. Suggested demonstration order

1. Explain the business scenario and node sequence.
2. Execute CUST-001 and show the successful final audit.
3. Execute CUST-002 and show the rejected branch.
4. Explain that this version is a simulation, and summarize the changes needed for production SQL integration.
