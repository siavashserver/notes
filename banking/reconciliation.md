---
title: Reconciliation
---

## Reconciliation Processes in Iran’s Banking System

### Overview of Reconciliation in Electronic Banking

At its core, reconciliation is the systematic matching, validation, and
settlement of transactional data between internal records (core banking) and
external systems (networks, switches, PSPs, and settlement infrastructures). The
process involves not only the technical collation and comparison of transaction
sets but also the management of exceptions, disputes, and adjustments, with an
overarching aim to eliminate discrepancies, ensure correct transfers of value,
and maintain compliance with national and international regulations.

**Key Reconciliation Touchpoints in Iran:**

- **ATM Transactions:** On-us (issuer and acquirer are same) and off-us
  withdrawals, balance inquiries, transfers
- **POS Terminal Transactions:** Merchant sales, refunds, and chargebacks
- **E-commerce Platform Transactions:** Online purchases, refunds, failures
- **Interbank Transfers:** Including real-time and batch payments
- **Switch Settlements:** Via Shetab (debit/credit card network) and Shaparak
  (PSP network)
- **Large Value Payments:** Via RTGS (Real Time Gross Settlement)

Across each of these domains, reconciliation serves to connect the dots between
four primary actors:

- **Acquirers:** Banks or institutions acquiring merchants’ payment transactions
- **Issuers:** Banks issuing cards or payment instruments to customers
- **Payment Service Providers (PSPs):** Technical intermediaries facilitating
  transaction routing, authentication, and settlement
- **Central Bank/Core Networks:** The Central Bank of Iran (CBI), plus Shetab
  and Shaparak switches, as central clearing and settling entities

**Figure 1: Core Reconciliation Flows in Iran**

| Transaction Type | Source Systems         | Target Systems       | Settlement Channel    | Primary Actors                    |
| ---------------- | ---------------------- | -------------------- | --------------------- | --------------------------------- |
| ATM (On-Us)      | ATM EJ, Core Banking   | CBS, ATM Switch      | Internal, Shetab      | Issuer bank                       |
| ATM (Off-Us)     | ATM EJ, Core Banking   | Shetab Switch, CBS   | Shetab, RTGS          | Issuer, Acquirer, CBI             |
| POS (On-Us)      | POS Terminal, CBS      | CBS, Shaparak Switch | Shaparak, Shetab      | Acquirer, PSP                     |
| POS (Off-Us)     | POS Terminal, Shaparak | Shetab, Merchant CBS | Shaparak, Shetab, CBI | Acquirer, Issuer, PSP, CBI        |
| E-commerce       | Website/PSP, CBS       | Shaparak, Shetab     | Shetab, Shaparak      | Acquirer, PSP, Issuer, CBI        |
| RTGS/Big value   | Core Banking, RTGS Sys | RTGS Sys, CBI Ledger | RTGS                  | Sending bank, Receiving bank, CBI |

_EJ: Electronic Journal; CBS: Core Banking System_

---

#### The General Reconciliation Lifecycle

1. **Initiation:** Transaction is performed at an endpoint (ATM, POS, Web) and
   logged in local (terminal) and host (bank) systems.
2. **Network Switching:** Transaction data is routed through relevant switches
   (Shetab/Shaparak) and, if applicable, to the issuer/acquirer for
   authorization.
3. **Authorization & Response:** Issuer bank validates funds/credit, returns a
   response to the acquirer via the switch.
4. **Transaction Recording:** All transaction steps are logged (transaction IDs,
   references, timestamps) at each actor.
5. **Batch Settlement & Clearing:** At predefined intervals (e.g., end-of-day),
   transaction data is exchanged in settlement files and clearing messages
   between the switch, PSPs, banks, and CBI.
6. **Automated/Manual Reconciliation:** Each bank/PSP system collates incoming
   and outgoing transaction records and matches them against internal logs;
   discrepancies are flagged for exception handling.
7. **Exception Management:** Errors, mismatches, or failures trigger specific
   remediation workflows (disputes, reversals, adjustment vouchers, customer
   communication).
8. **Final Settlement:** Once all positions agree, net settlements occur via
   interbank funding (RTGS or deferred net settlement through the Central Bank).
9. **Reporting and Compliance:** All steps recorded and reported for
   regulatory/audit purposes.

**Diagram A (see below): Iranian Transaction Reconciliation Ecosystem**

```
[Customer/Cardholder]
            |
        [ATM/POS/E-Comm Terminal]
            |
      [Acquirer/PSP]
            |
      [Shaparak/Shetab]
       /          \
[Issuer Bank]   [Acquirer Bank]
       \__________/
         (Settlement through CBI/RTGS)
```

---

## Systems and Entities Involved in Reconciliation

### 1. **Core Payment Networks: Shetab and Shaparak**

- **Shetab Network:** Iran’s national interbank switch for card transactions
  (since 2002), connects all banks, ATMs, and POS for debit/credit card
  processing. Handles both on-us and off-us transactions and is the ultimate
  clearinghouse for interbank card payments.
- **Shaparak:** Electronic Card Payment Network (est. 2012), acts as the
  regulating and operational authority for PSPs in Iran. All POS and
  internet-based payment flows are routed through Shaparak, which authenticates,
  authorizes, and monitors transaction compliance before passing relevant data
  to Shetab or reverse for settlement. Shaparak issued quality/security
  standards and enforces compliance for every PSP operation in Iran.

### 2. **Bank Switches and Internal Systems**

- **Core Banking Systems (CBS):** Each bank’s accounting and transaction ledger;
  all transaction entries are ultimately reconciled here.
- **ATM & POS Host Systems:** Serve as interfaces between the field devices and
  the bank’s core/network.
- **Electronic Journal (EJ):** Device-level logs recording all transaction
  details at ATMs, essential for dispute resolution and initial data matching.

### 3. **Payment Service Providers (PSPs)**

PSPs are licensed operators (regulated by Shaparak and the CBI) responsible for
provisioning POS devices, online payment gateways, and managing the initial
capture and technical routing of payment messages for e-commerce and retail
merchants. PSPs must collect and securely transmit transactional data in
required standard formats, maintain daily reconciliation logs, and act as
operational intermediaries between merchants, banks, and network operators.

### 4. **Acquirer and Issuer Banks**

- **Acquirer (Merchant’s Bank):** Accepts card payments for merchants, initiates
  transaction requests to the switch, and processes settlements into merchant
  accounts.
- **Issuer (Customer’s Bank):** Issues payment cards, authorizes transactions,
  and manages account-level reconciliation (debits, credits, reversals) for its
  cardholders.

### 5. **Central Bank of Iran (CBI)**

CBI is the final arbiter of settlement, issuing directives for money movement
between banks, and hosting the RTGS and ACH infrastructure for high-value and
bulk batch settlements. It oversees and enforces all major regulatory,
reporting, and compliance requirements for reconciliation, data retention, and
audit trails for all payment system participants.

---

## Reconciliation Flows and Detailed Examples

### ATM Transaction Reconciliation in Iran

#### On-Us vs Off-Us ATM Transactions

- **On-Us Transactions:** Cardholder uses an ATM of their own issuing bank. The
  transaction flows through internal bank systems, with reconciliation matching
  Electronic Journal entries, switch records, and CBS postings. Validation is
  typically simpler: a three-way match between EJ, Switch, and CBS.

- **Off-Us Transactions:** Cardholder uses an ATM owned by another bank.
  Transaction goes to the acquirer’s host, is routed to Shetab, which in turn
  reaches out to the issuer bank for authorization. Settlement positions between
  banks are updated and reconciled daily or in real-time.

**Typical ATM Reconciliation Steps:**

1. **Transaction Logging:** ATM records transaction in the electronic journal,
   and simultaneously, the switch and the CBS log events.
2. **Host & Switch Matching:** ATM host forwards data to the switch, which
   aggregates and formats records for interbank clearing with Shetab.
3. **Shetab Settlement:** Shetab’s central switch matches transaction logs from
   all participating banks and produces a clearing file detailing what each bank
   owes or is owed.
4. **Bank Reconciliation:** Each bank’s reconciliation team matches its internal
   CBS/host records with the Shetab clearing file and its ATM terminals’ EJs for
   each transaction.
5. **Reconciliation Exceptions:** Discrepancies (e.g., successful debit on CBS
   but failed cash dispense due to hardware fault) are flagged for investigation
   and adjustment.

**ATM Reconciliation Exception Scenarios:**

| Scenario                            | Nature of Exception              | Resolution                                      |
| ----------------------------------- | -------------------------------- | ----------------------------------------------- |
| Cash not dispensed, account debited | Customer debited, no cash issued | Investigation, manual/account reversal, dispute |
| Partial cash dispensed              | Mismatch in records for amount   | Adjustment voucher, notification to customer    |
| ATM journal missing record          | Missing or corrupted EJ file     | Cross-check with switch, initiate correction    |
| Transaction reversals not posted    | Reversal at switch but not bank  | Match logs, manual posting on CBS               |

---

### POS Terminal Transaction Reconciliation

**Flow Overview:**

1. **Transaction Initiation:** Cardholder swipes or taps card at merchant POS;
   data passed to PSP/acquirer’s host.
2. **Switch Routing:** If off-us, transaction goes from acquirer’s host via
   Shaparak to Shetab, routed to issuer bank for approval.
3. **Authorization Logging:** Authorization responses logged at all actors (POS,
   PSP, Switch, CBS).
4. **Settlement Files:** At end of business cycle, all transaction records are
   consolidated. Acquirer banks/PSPs send reports/transaction files to Shaparak;
   Shaparak collates records for clearing and Net Settlement Reporting.
5. **Network-Wide Reconciliation:** All actors perform data matching with
   network-clearing files to identify and resolve discrepancies.

**POS Reconciliation Exception Examples:**

| Problem Type                         | Impact                          | Action Taken                                                   |
| ------------------------------------ | ------------------------------- | -------------------------------------------------------------- |
| Duplicate/phantom transactions       | Customer or merchant complaints | Refund/correction and root-cause analysis                      |
| Chargebacks/refunds                  | Claims for returned goods       | Acquirer resolves with merchant, initiates settlement reversal |
| Settlement delays or missing credits | Merchant not credited timely    | Immediate review, correction, customer notification            |

Automated reconciliation software is widely adopted by Iranian institutions,
especially as transaction volumes have grown. Modern tools support multi-source
data ingestion (e.g., core banking, switch logs, PSP records), rule-based
matching, force-matching for edge cases, real-time dashboards, and automated
exception routing to reduce human error and speed up dispute resolution.

---

### E-commerce Platform Transaction Reconciliation

With the surge in Iranian online retail and digital commerce, platform
reconciliation blends online payment gateways, PSPs, and multi-party
settlements.

**Key Steps:**

- **Order Initiation:** Payment request initiated from merchant’s e-commerce
  platform to online PSP gateway (usually under Shaparak’s supervision).
- **Authorization Flow:** Routing mirrors offline POS, but the risk of
  non-settlement (e.g., failed delivery, merchant disputes, refund requests) is
  higher.
- **Settlement Coordination:** PSPs maintain reconciliation files for all online
  transactions, mapped against Shaparak-cleared records, merchant accounts, and
  Shetab files for interbank duties.
- **Multi-Party Matching:** Automated systems aggregate and match data from
  e-commerce platforms, PSPs, acquirers, and core banking for transactions,
  refunds, chargebacks, and commission accounting.

| E-commerce Matching Points         | What is Reconciled?                                       |
| ---------------------------------- | --------------------------------------------------------- |
| Merchant Order vs. Payment File    | Merchant’s order management vs. payment in PSP file       |
| PSP Gateway vs. Shaparak File      | Gateway approval logs match Shaparak-settled transactions |
| Shaparak vs. Shetab/net settlement | Interbank/cross-PSP settlement clearing                   |

**Key e-commerce reconciliation challenge:** Matching (and accounting for)
multi-currency/foreign currency flows, failed payments, refunds, payment
processor fees, and reconciliation of returns or chargebacks, in compliance with
internal and Shaparak/CBI reporting mandates.

---

### Shetab Network Settlement and Reconciliation

**Shetab acts as the central clearinghouse for interbank transaction flows,
handling the main reconciliation loop for all card-based transactions across all
member banks.**

**Process Flow:**

1. All member banks upload transaction batches (cleared and pending) to Shetab
   at defined intervals (typically end of each business day).
2. Shetab validates the integrity, formats, and checksums of incoming files.
3. System-wide reconciliation is performed—transaction-by-transaction—against
   individual bank submissions and switch logs.
4. Clearing positions are defined for each bank: net debit (amount owed to the
   network) or credit (amount receivable).
5. A comprehensive clearing and settlement file is produced and communicated to
   all participants; financial settlements (net movements) are executed via
   Central Bank accounts or RTGS as warranted.
6. Discrepancies or mismatches (e.g., duplicate, missing, or conflicting
   transactions) are reported back, triggering banks’ or PSPs’ exception
   handling procedures.

---

### Shaparak Switch Reconciliation Mechanisms

**As the regulatory and operational heart of Iran’s PSP market, Shaparak
oversees technical and compliance reconciliation for all POS and online payment
channels.**

- PSPs provide detailed, standardized transaction files to Shaparak, which
  aggregates, validates, and reconciles them across the broader network.
- Shaparak enforces real-time monitoring and flagging of abnormal (e.g.,
  fraudulent, anomalous, noncompliant) transaction trends, directly feeding into
  both clearing operations and regulatory oversight.
- End-of-period reconciliation files and compliance reports flow from Shaparak
  to banks, merchants, and the CBI for audit and settlement purposes.

---

### Iran RTGS System Reconciliation Flow

**RTGS is utilized primarily for the settlement of large-value interbank
payments and the backend clearing of net positions derived from Shetab,
Shaparak, and bulk ACH payments.**

**Key Steps:**

- High-value transaction instructions are processed individually in near
  real-time by the CBI’s RTGS platform. Each participating bank has a master
  account at the CBI; all debits and credits are tracked in real-time.
- At the end of each clearing cycle (for card/PSP transactions), the net
  multilateral positions established by Shetab or Shaparak are entered into the
  RTGS system, ensuring finality and irrevocability of settlement.
- Banks and the CBI reconcile these movements against internal ledgers and
  account statements to ensure all external and internal records align
  precisely.

---

## Multi-Party Reconciliation: Detailed Example Flow

Let’s walk through a typical multi-party (ATM off-us) reconciliation scenario in
Iran:

**Example: ATM Withdrawal on Another Bank’s Device (Off-Us Transaction)**

1. **Transaction Initiation:** Customer with a Bank A debit card initiates a
   withdrawal at Bank B’s ATM.
2. **ATM Records:** Bank B’s ATM logs the request in its electronic journal and
   sends transaction details to Bank B’s host.
3. **Switching:** Bank B’s host sends the transaction to Shetab, which
   identifies Bank A as the issuer.
4. **Authorization:** Shetab forwards the transaction to Bank A, which
   authorizes and debits the account if sufficient funds are available;
   authorization response is sent back via Shetab to Bank B.
5. **Cash Dispense and Logging:** Bank B’s ATM dispenses cash, records
   transaction completion in its EJ and notifies its host.
6. **Clearing Files:** At day’s end, both Bank A and Bank B upload their
   transaction logs to Shetab; Shetab reconciles all transactions, identifies
   net obligations, and constructs the clearing file.
7. **Reconciliation:** Each bank’s reconciliation system matches its ATM host
   and CBS records with the Shetab file. Any unmatched items (e.g., cash not
   dispensed but account debited) are flagged for manual review and customer
   remediation.
8. **Central Bank Settlement:** Net payments owed between banks are settled
   through the CBI, typically using RTGS for interbank funds transfer.

**Key Points:**

- Exception management is crucial for failed or mismatched transactions (e.g.,
  due to ATM hardware failures, communication losses).
- Customer disputes and reversals are logged, tracked, and resolved following
  strict CBI and network guidelines.
- Audit trails are preserved at every step for regulatory and dispute
  auditability.

---

## Error Handling Mechanisms in Iranian Payment Reconciliation

### Categories of Reconciliation Errors

- **Technical Failures:** Hardware errors, system downtime, communication losses
  leading to missing or duplicate transactions.
- **Business Logic Mismatches:** Timing differences, incomplete settlements,
  multi-currency issues.
- **Fraud/Anomaly Detection:** Suspicious or unauthorized transaction patterns,
  as flagged in automated monitoring systems.
- **Chargebacks and Disputes:** Customer-initiated complaints (e.g., non-receipt
  of goods, refund requests, card fraud).
- **Operational Errors:** Manual posting mistakes, delayed settlement postings,
  incorrect commission/fee calculation.

### Exception Management Workflow

- _Automatic Detection_: Modern reconciliation platforms utilize rule-based and
  increasingly AI-driven engines to automatically flag mismatches in real time.
- _Exception Queues_: Unmatched transactions enter exception management queues,
  prioritized by risk, value, and customer impact.
- _Investigation and Root Cause Analysis_: Designated back-office teams review
  exception items, examine logs, correlate with customer/merchant communication,
  and determine corrective actions.
- _Corrective Transactioning_: Reversals, refunds, or adjustment vouchers are
  posted as needed, with customer/merchant notification.
- _Audit and Root Cause Tagging_: Exception resolutions are coded and tagged,
  forming a database of resolution types for compliance reporting and process
  improvement.

---

## Compliance Requirements and Regulatory Mechanisms

### Central Bank of Iran (CBI) Oversight

- **Reporting Mandates:** Banks, PSPs, and networks are required to submit
  detailed reconciliation reports, exception logs, and compliance attestations
  to the CBI regularly.
- **Audit and Inspection:** CBI, in conjunction with network operators
  (Shetab/Shaparak), conducts periodic audits to ensure the integrity of
  reconciliation processes, data security, and compliance with anti-fraud and
  anti-money laundering protocols.
- **Safeguarding Requirements:** CBI mandates that all customer funds handled by
  PSPs and payment institutions are appropriately segregated and reconciled,
  with daily verification of safeguarding account balances.
- **Anti-Money Laundering (AML) and Countering the Financing of Terrorism
  (CFT):** Both real-time and post-settlement checks are required, and
  reconciliation logs serve as audit trails for suspicious activity monitoring.
- **Data Security Standards:** Both Shetab and Shaparak enforce international
  and national data protection and transaction security standards for all
  connected devices, networks, and databases.

### Recent Regulatory Developments

- Progressive adoption of Basel III standards, introducing liquidity, capital,
  and operational risk controls, which require enhancements in the granularity
  and timeliness of reconciliation and reporting by Iranian banks.
- Increased focus on automation and AI for regulatory reporting and anomaly
  detection in reconciliation workflows—enabling earlier detection of fraud,
  systemic errors, and compliance lapses.

---

## Summary and Main Takeaways

- **Iran’s banking system relies on an integrated, multi-layered reconciliation
  network spanning ATMs, POS terminals, e-commerce, Shetab, Shaparak, and CBI
  systems.**
- **Reconciliation is not a single process but a sequence of tightly coupled
  activities—data matching, exception management, settlement, and compliance
  assurance—within and across all transaction types and stakeholders.**
- **Shebat and Shaparak serve as the nerve centers for transaction validation,
  clearing, and compliance for all card and PSP flows, while the CBI’s RTGS
  system guarantees finality of high-value and network-wide settlements.**
- **Error and exception handling is elevated through AI-driven automation,
  structured workflows, and dynamic dashboards, reducing manual errors and
  expediting resolution for both operational mistakes and fraud.**
- **CBI and associated regulatory bodies enforce strict safeguarding, AML/CFT,
  data governance, and reporting standards, ensuring that all participants
  operate in a secure, auditable, and consumer-safe ecosystem.**
