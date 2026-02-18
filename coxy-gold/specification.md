# Coxy Wallet MVP Specification Document

**Product Name (recommended):** **CoxyFlow**
**Author:** Bernard Sibanda
**Company:** Coxygen Global
**Date:** **Wednesday, 18 February 2026**

## Table of Contents

1. Introduction
2. Product Vision and Naming
3. Scope and Core User Journeys
4. MVP Build Plan and Milestones
5. Platform Specifications and Architecture
6. Integrations Specifications
7. Security, Compliance, and Risk Controls
8. Non-Functional Requirements
9. Measurable Outcomes and OKRs
10. Glossary of Terms


## 1. Introduction

CoxyFlow is a next-generation wallet designed to make crypto usable in everyday life by combining **Cardano crypto wallet features** with a **fiat e-wallet**, **mobile money rails**, and **debit/credit card capabilities** in a single user experience. The goal is simple: a person should be able to receive crypto, cash in via mobile money, top up using a bank card, convert funds when needed, and spend instantly—without feeling like they are switching between different financial systems.

This document defines the MVP scope, technical and operational specifications, integration requirements, and measurable outcomes to deliver a real-world seamless payment experience.

## 2. Product Vision and Naming

### 2.1 Vision

CoxyFlow bridges two worlds:

* **Blockchain value** (Cardano ADA + tokens, self-custody, transparency, global transfers)
* **Everyday money** (mobile money cash-in/out, card top-ups, merchant spending, dispute handling)

The experience must feel like modern fintech: fast, clear balances, predictable fees, and reliable payment status updates.

### 2.2 Recommended Name: CoxyFlow

**CoxyFlow** communicates smooth movement of money across rails—crypto ↔ fiat ↔ mobile money ↔ cards. It is short, brandable, and aligns with the product promise: “money that flows wherever the user needs it.”

Suggested tagline options:

* **CoxyFlow — One Wallet. All Money.**
* **CoxyFlow — Crypto to Cash, Instantly.**
* **CoxyFlow — Spend Crypto Anywhere.**

## 3. Scope and Core User Journeys

To deliver a “real-world seamless” wallet, the MVP must support the following user journeys end-to-end.

### 3.1 Onboarding and Account Setup

A user installs the app and creates a crypto wallet (non-custodial keys on device). In parallel, the user completes KYC to activate the fiat e-wallet and payment rails. The system should guide users through identity verification using a partner sandbox in the MVP phase, preparing for production rollout with the same workflow.

### 3.2 Add Money (Cash-in)

Users must be able to fund their wallet through multiple entry points:

* **Crypto deposit**: receive ADA and tokens into their Cardano address.
* **Mobile money cash-in**: initiate a request-to-pay (e.g., STK push where applicable), confirm payment, and see funds reflected in the fiat balance once confirmed.
* **Card top-up**: top up fiat balance using debit/credit cards with 3DS authentication enabled for security.

This multi-rail cash-in capability ensures users can start using the wallet even if they don’t already hold crypto.

### 3.3 Send and Receive Value

CoxyFlow should support sending value in the form users expect:

* **Crypto transfers**: send ADA/tokens to Cardano addresses, with clear fee display and confirmation progress.
* **Fiat transfers (internal/P2P)**: send fiat balance to another user or via phone/email mapping where supported by the partner.
* **Mobile money send-out**: withdraw fiat to a mobile money number, with clear transaction states and completion notifications.

### 3.4 Spend in the Real World

Spending must work reliably and transparently:

* Users can spend fiat using a **virtual card** (and later a physical card), with transactions appearing instantly in history.
* If fiat is insufficient, the wallet can **auto-convert** crypto to fiat (with user permission and clear rate/fees) to complete the purchase.
* Holds (authorizations) must be represented accurately: “Available vs Pending vs On hold”.

### 3.5 Conversion and Pricing Transparency

A key differentiator is conversion that feels safe:

* When a conversion is needed, the wallet provides an FX quote, shows fees/spread, and applies an expiry window (e.g., 30 seconds).
* The user must always understand: **rate**, **fee**, **final amount received**, and **why** a conversion happened (manual or auto-convert rule).

## 4. MVP Build Plan and Milestones

The MVP is delivered in milestone increments that progressively unlock the full “seamless” experience:

1. **Crypto wallet core**: Cardano send/receive, token balances, transaction history.
2. **Fiat ledger + KYC**: partner sandbox integration, fiat balances, transaction posting.
3. **Mobile money cash-in/out**: aggregator integration, webhook status updates, wallet balance updates.
4. **Card top-ups (3DS enabled)**: card funding into fiat ledger with secure authentication.
5. **Auto-convert + FX quotes**: quote service, expiry, user consent, execution, posting.
6. **Virtual card issuing + disputes + holds**: authorization holds, clearing, reversal flows, disputes workflow.
7. **Reconciliation automation + risk rules v1**: daily matching, exception queues, basic fraud/AML triggers.

This milestone approach ensures the wallet remains testable and stable at each stage, reducing integration risk.

## 5. Platform Specifications and Architecture

CoxyFlow requires a hybrid architecture: **local crypto custody on device** plus **backend services** that orchestrate fiat/payment operations and reporting.

### 5.1 Client Applications

The MVP should prioritize a **mobile app** as the primary interface. It must support:

* Key management and transaction signing prompts for crypto operations.
* A unified dashboard showing crypto and fiat balances with clear labeling (Available/Pending/Hold).
* Payment initiation flows for mobile money and card top-up.
* Notifications for transaction state changes.

A browser extension can exist separately, but the MVP experience is best delivered on mobile first.

### 5.2 Core Backend Services

The backend exists to provide reliability, orchestration, and compliance for fiat/payment rails:

* **Identity & KYC service**: onboarding, document verification workflow, KYC tiering.
* **Fiat ledger service (double-entry)**: the accounting source of truth for all fiat balances and holds.
* **Payments orchestration service**: handles mobile money, card top-ups, issuing flows, webhooks, retries.
* **FX/Quotes service**: provides rate calculation, slippage/expiry handling, and conversion execution hooks.
* **Risk & fraud engine (v1)**: rules, velocity checks, suspicious pattern detection.
* **Reconciliation service**: compares internal ledger with partner/aggregator settlement reports; flags exceptions.
* **Notifications service**: push/SMS/email updates driven by payment state transitions.

### 5.3 Reliability Principles

Because payment rails are asynchronous and webhook-driven, the platform must implement:

* **Idempotency keys** for every payment initiation to prevent double-charging.
* **Webhook verification** (signature checking) and safe retry processing.
* **State machines** for transactions so the UI always matches reality.
* **Audit trails** for compliance, disputes, and incident investigations.

## 6. Integrations Specifications

### 6.1 Mobile Money Integration (Aggregator)

The MVP should integrate via an aggregator first to reduce complexity and accelerate time-to-market. The integration must support:

* **Cash-in initiation** (request-to-pay): returns a reference + pending status.
* **Cash-out/disbursement**: sends funds to a mobile number and returns transaction reference.
* **Status webhooks**: updates transaction outcome (success/failure/expired).
* **Reconciliation artifacts**: daily transaction exports or report endpoints for matching.

A production-ready design assumes webhooks may arrive late, out-of-order, or duplicated, so the wallet must reconcile transaction states deterministically.

### 6.2 Card Top-Ups (3DS Enabled)

Card top-ups are handled through a PSP/acquirer using tokenized card handling (no raw card data stored in CoxyFlow systems). The MVP must include:

* **3DS challenge flows** when required.
* Immediate posting to a pending state, then finalization on provider confirmation.
* Clear user messaging when top-up is pending vs complete vs failed.

### 6.3 Auto-Convert + FX Quotes

Conversion is what makes spending seamless. The MVP must include:

* A quote endpoint returning: rate, spread, fees, expiry timestamp, and final amounts.
* A conversion execution endpoint that posts:

  * crypto sell (or swap) event,
  * fiat credit to ledger,
  * fee postings,
  * and a link to the original intent (e.g., card purchase funding).

### 6.4 Virtual Card Issuing + Disputes + Holds

Card issuing introduces additional complexity that must be reflected in the ledger:

* **Authorization holds** reserve balance instantly.
* **Clearing/settlement** finalizes or adjusts the hold.
* **Reversals/voids** release held funds.
* **Disputes** open a case, attach evidence, track timelines, and apply ledger holds/chargeback postings.

The MVP should support a minimal but complete dispute lifecycle: open → under review → resolved, with complete audit logging.

## 7. Security, Compliance, and Risk Controls

### 7.1 Crypto Key Security (Non-Custodial)

CoxyFlow should generate keys on device and protect them using:

* Encrypted storage in platform secure hardware (where available).
* Biometric unlock + passcode fallback.
* Strong re-authentication for seed phrase viewing/export.
* Optional encrypted backup (future enhancement).

### 7.2 Payments Security (Fiat + Cards)

The MVP should avoid handling raw card data directly by using PSP tokenization and hosted components. Additionally:

* Device binding and session controls reduce account takeover risk.
* Rate limiting, anomaly detection, and login protections reduce fraud.

### 7.3 Risk Rules v1 (Practical Baseline)

The initial risk engine should detect:

* Unusual velocity (many top-ups/cash-outs in short time).
* Repeated failed attempts (possible fraud testing).
* Mismatched identities (KYC name vs card/mobile money patterns where available).
* Suspicious circular flows (cash-in then immediate cash-out repeatedly).

The goal is not perfect fraud prevention in MVP, but a workable baseline with clear escalation paths.

## 8. Non-Functional Requirements

For a “seamless” experience, performance and reliability matter as much as features.

* **Latency:** balance refresh should feel instant; confirmations should update quickly after provider responses.
* **Availability:** services must degrade gracefully when partners are down (show pending states, retry safely).
* **Scalability:** webhook bursts must not break processing; queue-based designs are recommended.
* **Observability:** every transaction must be traceable end-to-end via a single reference ID, with logs and metrics.

## 9. Measurable Outcomes and OKRs

This section converts the MVP milestones into measurable outcomes suitable for tracking delivery and readiness.

### 9.1 Five Measurable Outcomes (from the MVP plan)

1. **Crypto wallet core is production-ready**
   Success is measured by high transaction success rates in staging and correct balance/history representation compared to the indexer.

2. **Fiat ledger + KYC works end-to-end in partner sandbox**
   Success is measured by KYC completion rate and strict double-entry ledger correctness with zero imbalance.

3. **Mobile money cash-in/out is reliable and reconcilable**
   Success is measured by time-to-final-state, webhook success rate, and low unmatched transaction rate in daily reconciliation.

4. **Card top-ups are secure and performant (3DS enabled)**
   Success is measured by completion rate, zero sensitive card storage, and timely balance updates.

5. **Spend experience is complete (FX + auto-convert + virtual card + controls)**
   Success is measured by fast quotes, fast hold creation, complete dispute audit trails, and reconciliation/risk performance targets.

### 9.2 OKR Table (8-week MVP plan)
Here’s a clean **Markdown list version** you can paste to replace **Section 9.2** (no table):

## 9.2 OKR List (8-week MVP plan)

**Objective:** Ship a CoxyFlow MVP that enables **crypto + fiat + mobile money + cards** with a seamless and safe spend experience.

### KR1 — Crypto wallet core is stable (Cardano send/receive, tokens, history)

* **Target metrics (pass/fail):**

  1. ≥ **95%** successful send/receive across **500** staged transfers
  2. Balances + history match indexer within **±1 block**
  3. Crash-free session rate ≥ **99.5%**
* **Owner:** Wallet/Blockchain Engineer
* **Target date:** **04 Mar 2026**

### KR2 — Fiat ledger + KYC works end-to-end (partner sandbox)

* **Target metrics (pass/fail):**

  1. ≥ **90%** KYC completion rate for **200** test signups
  2. **0 ledger imbalance** (debits=credits) over **1,000** postings
  3. Balance update latency ≤ **5s** after provider confirmation
* **Owner:** Backend/Payments + Compliance
* **Target date:** **18 Mar 2026**

### KR3 — Mobile money cash-in/out is reliable (aggregator)

* **Target metrics (pass/fail):**

  1. ≥ **92%** reach final status (SUCCESS/FAILED/EXPIRED) within **2 min**
  2. Webhook processing success ≥ **99%**
  3. Daily unmatched transactions in reconciliation ≤ **0.5%**
* **Owner:** Payments Integration Engineer
* **Target date:** **01 Apr 2026**

### KR4 — Card top-ups are secure and performant (3DS enabled)

* **Target metrics (pass/fail):**

  1. ≥ **90%** successful top-ups where 3DS is available
  2. **100%** tokenized/hosted fields (no PAN stored)
  3. Balance reflects provider result within **≤10s**
* **Owner:** Payments Engineer + Security
* **Target date:** **01 Apr 2026**

### KR5 — Seamless spend + controls are live (FX, auto-convert, virtual card, disputes/holds, recon + risk v1)

* **Target metrics (pass/fail):**

  1. FX quote response **p95 ≤ 1s** with expiry (e.g., **30s**)
  2. Holds created within **≤2s** for **95%** authorizations
  3. Disputes tracked with **100%** audit trail coverage
  4. Reconciliation drift ≤ **0.2%** daily
  5. Risk rules catch ≥ **80%** seeded fraud with false positives ≤ **5%**
* **Owner:** Product + Payments + Risk
* **Target date:** **15 Apr 2026**

## 10. Glossary of Terms

* **3DS (3-D Secure):** A card payment authentication method that reduces fraud by requiring additional verification from the cardholder.
* **Acquirer:** A financial institution/processor that handles card payments for merchants.
* **Aggregator (Mobile Money):** A provider that offers a unified API across multiple mobile network operators (MNOs) for cash-in/out.
* **Authorization Hold:** Temporary reservation of funds when a card purchase is approved but not yet settled.
* **Chargeback:** A card payment reversal initiated by the cardholder’s bank, usually after a dispute.
* **Clearing/Settlement:** The final processing stage where a transaction is confirmed and funds move between institutions.
* **Double-entry Ledger:** An accounting system where every transaction records an equal debit and credit to keep books balanced.
* **FX Quote:** A price offered to convert one currency/asset to another, usually time-limited.
* **Idempotency Key:** A unique key used to ensure repeated requests (retries) do not create duplicate transactions.
* **KYC (Know Your Customer):** Identity verification process required for regulated financial services.
* **MNO (Mobile Network Operator):** A telecom provider that often runs mobile money services.
* **Non-custodial Wallet:** A wallet where the user controls the private keys (the provider does not hold them).
* **PSP (Payment Service Provider):** A company that enables card payments and other payment methods via APIs.
* **Reconciliation:** Comparing internal records against partner/provider reports to ensure balances and transactions match.
* **Risk Rules v1:** The first version of fraud and abuse detection rules (e.g., velocity checks, patterns, anomalies).
* **STK Push:** A mobile money request-to-pay flow initiated via the phone SIM toolkit (common in some markets).
* **Tokenization (Cards):** Replacing sensitive card data with a token so systems don’t store raw PAN/CVV.

