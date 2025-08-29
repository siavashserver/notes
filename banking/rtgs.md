---
title: RTGS
---

## RTGS: Real-Time Gross Settlement System

### A. RTGS Overview

A Real-Time Gross Settlement (RTGS) system facilitates the immediate,
irrevocable, and individual settlement of large-value or time-urgent electronic
payment instructions between banks. Unlike net settlement—where obligations are
aggregated and settled in bulk at the end of a cycle—RTGS processes each
transaction singularly and in real time, drastically lowering systemic and
settlement risks and providing real-time liquidity and transparency in the
monetary system.

**Key RTGS Characteristics:**

- **Settlement Basis:** Individual (not batched), processed on order receipt.
- **Timing:** Immediate (“real-time”) finality.
- **Settlement Finality:** Irreversible on completion.
- **Participants:** Commercial banks and select licensed non-bank financial
  institutions.
- **Value Focus:** Predominantly high-value, interbank, and wholesale
  transactions.
- **Risk Profile:** Eliminates settlement risk (Herstatt risk) and reduces
  credit/counterparty risk.

RTGS systems are typically owned and operated by a country’s central bank, in
Iran’s case, the Central Bank of Iran (CBI), which ensures that all transactions
are settled in central bank money.

---

### B. RTGS in Iran: Implementation and Operation

#### 1. Historical Context and Progression

The implementation of Iran’s RTGS, locally known as **SATNA (سامانه تسویه ناخالص
آنی)**, is a core component of the broader national payment system modernization
project that began in the early 2000s. The project aimed to digitize and
synchronize all interbank payments, establish an integrated framework for
high-value settlements, and facilitate seamless real-time money transfers
nationally.

The rollout of RTGS in Iran unfolded in multiple phases:

- **First phase:** Daily settling of Shetab Center operations.
- **Second phase:** Settlement for checks and banking documents through the
  clearing house.
- **Third phase:** Bank-to-bank, large-value settlements.
- **Fourth phase:** Customer-to-customer real-time settlements.

By transitioning from cumbersome, paper-based, branch-dependent cash transfers
to a robust, electronic, centralized RTGS, the CBI radicalized the speed,
safety, and efficiency of Iran's financial operations.

#### 2. Architecture and Transaction Flow

Iran’s RTGS is designed as a centralized switching and settlement hub:

- **Participants** (such as banks) connect and submit payment instructions
  directly to the Central Bank.
- **Settlement** occurs instantly by debiting the originating bank’s reserve
  account and crediting the beneficiary’s bank reserve account at the Central
  Bank.
- _No_ physical cash is exchanged; only ledger entries change.

The system is supported by redundant communication lines (VSAT-based MQ
messages—chosen due to intermittent access to SWIFT under sanctions), advanced
security layers, and strict operational controls.

#### 3. Core Functions of SATNA

- **Large-Value Transfers:** SATNA is primarily used for significant interbank
  payments. It also enables high-value corporate or government transactions.
- **Immediate Finality:** Transactions are irrevocable and instantly settled at
  the central bank level.
- **Systemic Risk Mitigation:** Direct settlement reduces chain-reaction risk
  inherent in deferred/net settlement systems.
- **Liquidity Management:** Offers banks real-time visibility into their
  liquidity and positions—a critical component for monetary operations.

#### 4. Interoperability and Integration

RTGS in Iran:

- **Integrates with Shetab:** Settlement of Shetab interbank transactions at the
  end of each day is carried out through SATNA.
- **Connects with ACH, SSSS, PAYA, and Shaparak:** SATNA forms the backbone,
  while PAYA (Automated Clearing House) handles bulk retail settlement, and
  Shaparak handles POS and online gateway settlements, all ultimately
  interfacing with SATNA for final high-value clearing.
