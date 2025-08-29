---
title: Shetab
---

## Shetab: Interbank Information Transfer Network

### A. Shetab System Overview

**Shetab (شتاب, "Acceleration")**, officially known as the Interbank Information
Transfer Network, is Iran’s national card-based clearing and automated payments
system, launched in 2002. The creation of Shetab revolutionized card
interoperability, mandating unified standards for all card-issuing banks and
their ATMs, POS, and online systems.

**Shetab’s Key Stats:**

- **Member Banks:** Virtually every Iranian commercial and specialized bank.
- **Network Reach:** 57,000+ ATMs (2019), millions of POS devices via PSPs.
- **Transaction Volume:** Hundreds of millions of ATM, POS, and card-to-card
  transactions monthly.
- **Geographic Expansion:** Direct interoperability with Russia (MIR), China
  (UnionPay), Qatar (NAPS), Bahrain, Kuwait, and ongoing connections set with
  other Asia-Pacific, Middle-East, and BRICS regions.

---

### B. Functions and Services

#### 1. Transaction Types Supported

- **ATM Withdrawals:** Any Shetab-enabled card can be used at any connected ATM.
- **EFTPOS:** Purchases, cash advances, bill payments, and balance inquiries at
  POS across banks.
- **Card-to-card Transfer:** Both intra- and inter-bank card transfer, including
  digital Sahab service for instant funds transfer.
- **Online Transactions:** Unified standard for Iranian payment gateways.
- **Cross-Border Transactions:** Cooperation with foreign banks, especially in
  the Gulf and recently, Russia.

#### 2. Standardization and Interoperability

Before Shetab, each Iranian bank operated siloed card systems that only worked
on that bank’s hardware. Shetab imposed a unified card data structure, routing
logic, transaction codes, and settlement process, making it mandatory that every
debit/credit card be Shetab-capable.

#### 3. Connectivity and Internationalization

Beyond Iran’s borders, Shetab is being connected to foreign payment
networks—facilitating cross-border travel and commerce particularly important
under international sanctions. The most recent breakthrough is its full
technical and operational integration with Russia’s MIR network—the largest such
bilateral payments interoperability project outside the SWIFT regime.

---

## Shetab 7 vs. Shetab 8: Architectural Comparison

### A. Overview of Generational Shifts

Shetab’s architectural evolution mirrors global trends in transaction switching:
increasing transaction velocity, lowering latency, bolstering security, and
improving international interoperability. The transition from **Shetab 7** to
**Shetab 8** over the last several years marked a major generational leap.

---

### B. Architectural Differences

| Feature                   | Shetab 7                                                      | Shetab 8                                                          |
| ------------------------- | ------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Core Protocol Stack**   | Traditional financial messaging (ISO 8583-based)              | Modern messaging (ISO 20022 for enhanced data support)            |
| **Processing Engine**     | Centralized monolithic switch                                 | Modular, microservices-based distributed architecture             |
| **Hardware Platform**     | Virtualized x86 clusters, classic mainframe switching         | Distributed clusters, ARMv8 and virtualized ARM, cloud-adaptive   |
| **Security Layer**        | Legacy HSM integration, limited hardware-based isolation      | Multilayer HSM, secure enclave support, remote HSM management     |
| **Transaction Routing**   | Rule-based, static routing                                    | Dynamic, AI-assisted/metadata-driven routing                      |
| **International Interop** | Limited message adapter for foreign standards, custom mapping | Native multi-standard, deep support for MIR, UnionPay, JCB, etc.  |
| **Resilience/Recovery**   | Manual failover, active/passive clusters                      | Automated failover, active/active, cloud-native auto-scaling      |
| **Monitoring/Ops**        | SNMP, periodic manual reporting                               | Real-time telemetry, API analytics, deep audit logs               |
| **Compliance**            | PCI DSS v3, ISO 8583, Iran-specific CBI guidelines            | PCI DSS v4+, ISO 20022, GDPR-ready, advanced AML/CFT monitoring   |
| **API Connectivity**      | SOAP/XML for legacy bank integration                          | RESTful, GraphQL, and custom APIs for FinTech, open banking       |
| **Throughput**            | 2,000–10,000 TPS per cluster                                  | 10,000–50,000+ TPS per distributed node, horizontally scalable    |
| **Innovation Support**    | Scripted plug-ins, hard to extend                             | Dynamic module loading, support for tokenization, digital wallets |

---

#### Shetab 7 (Legacy Architecture)

Shetab 7’s architecture was built around a central, high-availability switch—a
design suited for the transactional volumes and regulatory landscape of the
mid-2010s. Transaction routing was rule-based, with bank-specific and
device-specific configuration tables. Security heavily relied on legacy HSMs,
with limited support for remote management or dynamic key lifecycle controls.

Scalability, though robust for its time, was ultimately limited by vertical
scaling (adding more power to a central core), and system upgrades required
significant planned outages.

---

#### Shetab 8 (Modern Distributed Architecture)

Shetab 8 embraces a modular, horizontal-scaling architecture inspired by
microservices and cloud-native principles:

- **Modularization** allows the system to respond to traffic peaks, updates, or
  failures by seamlessly distributing or re-routing loads across a mesh of
  switching nodes.
- **Enhanced Security**, through the deployment of advanced HSMs supporting
  remote attestation, physical tamper detection, and advanced multiparty key
  management.
- **Dynamic Routing** algorithms use transaction metadata, risk scoring, and
  real-time analytics to optimize transaction pathing—improving speed and
  reliability.
- **Native International Interoperability:** Direct support for complex,
  multi-standard cross-border payments, now essential for Russia MIR and future
  BRICS integrations.

---

### C. Routing and Interoperability Impact

**Transaction Routing Advancements:**

- **Shetab 8** employs AI-driven and rules-based real-time decisioning,
  automatically selecting the optimal network path, payment scheme, and
  compliance checks for each transaction. This is vital for international
  payments, which may touch Shetab, MIR, and even UnionPay or JCB (Japan Credit
  Bureau).
- **Resilience:** Shetab 8’s architecture allows for node isolation: if one
  datacenter or cloud cluster is compromised or overloaded, others take over
  with zero downtime.

**Interoperability:**

- Directly supports advanced fields and message schemas required for
  international clearing.
- API extensions allow FinTechs and partner banks to integrate securely and
  rapidly—facilitating the rise of wallet apps, tokenized payments, and
  cross-border B2B.

**Compliance & Analytics:**

- Under the new regime, transaction-level auditing, real-time fraud detection,
  and anti-money laundering measures are significantly more effective.
- Shetab 8 enables CBI and participating banks to respond to regulatory changes
  with configuration rather than hardware or code changes.

---

### D. Architectural Diagrams

#### Shetab 7 High-Level Logical Diagram

```
+-------------------+
| ATM / POS / Web   |---+
+-------------------+   |
                        v
+-------------------------+
|   Legacy Bank Switch    | (Bank-internal)         +----------+
+-------------------------+                         |  HSM     |
           |                                        +----------+
           v
+-------------------------+
|   Central Shetab Switch | <==>   HSM              (Central)
+-------------------------+
           |
           v
+-------------------+
| Settlement Engine |
+-------------------+
```

#### Shetab 8 High-Level Logical Diagram

```
+-------------------+
| ATM / POS / Web   |---+
+-------------------+   |
                        v
+-------------------------------+          +-----------+
| Distributed Bank/FinTech Gateway|------->| Clustered |
+-------------------------------+  |       |   HSM     |
                                   |       +-----------+
                                   v
+------------------------------------------+
| Shetab 8 Distributed Switch Mesh         |
+----+-----------+------------+-----+------+
     |           |            |     |
+-----+   +-------+    +--------+  ... more clusters
|Node1|   |Node2  |    |Node N  |
+-----+   +-------+    +--------+
     |           |           |
     v           v           v
+------------------------------+
| Dynamic Routing & Analytics  |
+------------------------------+
     |
     v
+-------------------+
| Settlement Engine |
+-------------------+
```

_Each distributed node can be cloud-based, on-premises, or hybrid, and each has
direct, secure HSM connectivity._

---

### E. Impact on Payment System Integration

- **Scalability**: Shetab 8 supports Iranian payment volumes that continue to
  break records as more commerce moves online and cross-border.
- **Resilience**: Banks, merchants, and governments can trust that transactional
  continuity is assured even in the case of cyberattacks, physical attacks, or
  large-scale failures.
- **Speed**: Onboarding new payment schemes, wallets, and cross-border corridors
  is now measured in days or weeks rather than the months or years of Shetab 7.
