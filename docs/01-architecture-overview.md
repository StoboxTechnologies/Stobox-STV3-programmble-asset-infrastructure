# Architecture Overview

> Programmable asset infrastructure for institutional-grade tokenization

---

## Introduction

The **Stobox STV3 Protocol** is a programmable asset infrastructure designed for issuing and managing real-world financial instruments on-chain with embedded compliance, identity, governance, and data flows. It brings institutional rigor to tokenized assets by ensuring that every action—transfers, distributions, redemptions, governance, and data updates—is enforced through smart-contract logic rather than manual supervision.

STV3 is engineered for **enterprises, funds, corporates, commodity issuers, and regulated market participants** who require both the flexibility of blockchain technology and the structure of financial regulation. It enables organizations to issue assets that behave predictably, update automatically, and maintain regulatory alignment across jurisdictions.

---

## Platform at a Glance

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      STOBOX STV3 PROTOCOL                                    │
│                                                                              │
│   "Programmable Infrastructure for Real-World Assets"                       │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌─────────────┐
                              │   ISSUER    │
                              │  (Company)  │
                              └──────┬──────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
                    ▼                ▼                ▼
            ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
            │   CREATE    │  │    SELL     │  │   MANAGE    │
            │   TOKEN     │  │   TOKENS    │  │  INVESTORS  │
            └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
                   │                │                │
                   ▼                ▼                ▼
            ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
            │   Factory   │  │  Offering   │  │  StoboxDID  │
            │             │  │  Registry   │  │             │
            └─────────────┘  └─────────────┘  └─────────────┘
                   │                │                │
                   └────────────────┼────────────────┘
                                    │
                                    ▼
                           ┌───────────────┐
                           │ SECURITY      │
                           │ TOKEN (STV3)  │
                           │ + TREASURY    │
                           └───────────────┘
                                    │
                                    ▼
                           ┌───────────────┐
                           │  INVESTORS    │
                           └───────────────┘
```

---

## Core Components

The platform consists of four interconnected smart contract systems:

| Component | Purpose | Role in Platform |
|-----------|---------|------------------|
| [**StoboxRWAVaultFactory**](./components/vault-factory.md) | Token Creation | Deploys new security tokens with their treasuries |
| [**StoboxProtocolSTV3**](./components/protocol-stv3.md) | Security Token | The token itself with compliance and trading features |
| [**StoboxRWAOfferingRegistry**](./components/offering-registry.md) | Sales Management | Manages STOs, processes purchases, enforces rules |
| [**StoboxDID**](./components/stobox-did.md) | Identity Layer | Verifies investors, stores KYC data securely |

---

## Component Interaction

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        COMPONENT INTERACTION MAP                             │
└─────────────────────────────────────────────────────────────────────────────┘


     ┌──────────────────────────────────────────────────────────────────┐
     │                    StoboxRWAVaultFactory                         │
     │                    ─────────────────────                         │
     │              • Creates security tokens                           │
     │              • Deploys treasury contracts                        │
     │              • Manages token packages/versions                   │
     └────────────────────────────┬─────────────────────────────────────┘
                                  │
                                  │ creates
                                  ▼
     ┌──────────────────────────────────────────────────────────────────┐
     │                    StoboxProtocolSTV3                            │
     │                    ──────────────────                            │
     │              • ERC-20 security token                             │
     │              • Role-based access control                         │
     │              • Transfer validation                               │
     │              • Token issuance & redemption                       │
     │                         +                                        │
     │                    STV3 Treasury                                 │
     │              • Holds tokens and payments                         │
     │              • Manages reserves                                  │
     └──────────┬─────────────────────────────────────────┬─────────────┘
                │                                         │
                │ registers & sells                       │ validates
                ▼                                         ▼
     ┌─────────────────────────────┐       ┌─────────────────────────────┐
     │  StoboxRWAOfferingRegistry  │       │         StoboxDID           │
     │  ─────────────────────────  │       │         ─────────           │
     │  • Creates offerings (STOs) │       │  • Stores investor identity │
     │  • Processes purchases      │◄─────►│  • KYC/AML verification     │
     │  • Enforces investment rules│       │  • Privacy-preserving data  │
     │  • Manages lockups          │       │  • Multi-wallet support     │
     │  • Handles soft cap/refunds │       │  • Attribute management     │
     └─────────────────────────────┘       └─────────────────────────────┘
```

---

## Data Flow

### Token Creation Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TOKEN CREATION FLOW                                  │
└─────────────────────────────────────────────────────────────────────────────┘

  ISSUER                    FACTORY                     RESULT
    │                          │                           │
    │  1. Request token        │                           │
    │     (name, symbol,       │                           │
    │      supply, package)    │                           │
    │─────────────────────────►│                           │
    │                          │                           │
    │                          │  2. Deploy STV3 Token     │
    │                          │─────────────────────────► │ Token Contract
    │                          │                           │
    │                          │  3. Deploy Treasury       │
    │                          │─────────────────────────► │ Treasury Contract
    │                          │                           │
    │                          │  4. Link & Configure      │
    │                          │─────────────────────────► │ Ready to Use
    │                          │                           │
    │  5. Token addresses      │                           │
    │◄─────────────────────────│                           │
    │                          │                           │
```

### Investment Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          INVESTMENT FLOW                                     │
└─────────────────────────────────────────────────────────────────────────────┘

  INVESTOR          STV3 TOKEN         OFFERING REGISTRY        STOBOX DID
     │                  │                      │                     │
     │  1. Purchase     │                      │                     │
     │─────────────────►│                      │                     │
     │                  │  2. Get offering     │                     │
     │                  │─────────────────────►│                     │
     │                  │                      │  3. Verify DID      │
     │                  │                      │────────────────────►│
     │                  │                      │                     │
     │                  │                      │  4. Check rules     │
     │                  │                      │◄────────────────────│
     │                  │                      │     (country,       │
     │                  │                      │      accreditation) │
     │                  │  5. Validation OK    │                     │
     │                  │◄─────────────────────│                     │
     │                  │                      │                     │
     │                  │  6. Transfer payment │                     │
     │                  │  ← from investor     │                     │
     │                  │  → to treasury       │                     │
     │                  │                      │                     │
     │                  │  7. Transfer tokens  │                     │
     │  8. Receive      │  ← from treasury     │                     │
     │     tokens       │  → to investor       │                     │
     │◄─────────────────│                      │                     │
     │                  │  9. Record purchase  │                     │
     │                  │─────────────────────►│                     │
     │                  │                      │                     │
```

---

## Architectural Foundations

The protocol is built on three core pillars:

### 1. ERC-20 Compatibility via Diamond Standard (EIP-2535)

The STV3 Protocol combines **universal EVM compatibility** with **institutional-grade control and extensibility**. Each STV3 asset is exposed as an **ERC-20–compatible token**, ensuring seamless interoperability with wallets, custodians, indexers, analytics providers, and regulated infrastructure.

| Technology | Purpose |
|------------|---------|
| **Solidity 0.8.28+** | Smart contract language |
| **Diamond Pattern (EIP-2535)** | Modular, upgradeable contracts |
| **ERC-20** | Token standard compatibility |
| **[ERC-7943 (uRWA)](https://eips.ethereum.org/EIPS/eip-7943)** | Universal Real World Asset Interface |
| **OpenZeppelin** | Battle-tested security libraries |

#### ERC-7943 Compliance

STV3 tokens implement the **Universal Real World Asset (uRWA) Interface** ([ERC-7943](https://eips.ethereum.org/EIPS/eip-7943)), providing standardized compliance functions that enable DeFi protocols and applications to interact with tokenized real-world assets in a unified way:

| Function | Purpose | STV3 Implementation |
|----------|---------|---------------------|
| `canTransact(account)` | Check if account is allowed to transact | DID verification + KYC status |
| `canTransfer(from, to, amount)` | Check if transfer is allowed | Validation Engine rules |
| `getFrozenTokens(account)` | Get frozen token amount | LockupFacet |
| `setFrozenTokens(account, amount)` | Freeze/unfreeze tokens | `lockTokens()` in LockupFacet |
| `forcedTransfer(from, to, amount)` | Regulatory transfer | Recovery & legal compliance |

This compliance ensures that STV3 tokens can seamlessly integrate with any **DeFi protocol**, **AMM pool**, or **lending platform** that supports ERC-7943, while maintaining full regulatory compliance.

### 2. Decentralized Identity (DID) Layer

Every participant interacting with an STV3 asset is linked to a **Stobox DID**. This ensures that the token always knows:
- Who is allowed to hold or receive the asset
- Which investor category they belong to
- Whether they pass jurisdiction-specific requirements
- Whether they are subject to sanctions, limits, or restrictions

DID verification transforms assets from anonymous blockchain tokens into **identity-bound financial instruments**, supporting full regulatory traceability.

### 3. Validation Engine

The **validation engine** is the heart of programmable compliance. Before any transfer or lifecycle action occurs, the protocol checks:
- Jurisdictional rules
- Investor eligibility
- Lockups and vesting
- Whitelist or blacklist status

Transfers that violate rules never execute, meaning **non-compliant behavior becomes technically impossible**.

### Diamond Pattern Benefits

The platform uses the **Diamond Standard (EIP-2535)** for its core contracts, providing:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DIAMOND PATTERN                                       │
└─────────────────────────────────────────────────────────────────────────────┘

  Traditional Contract              Diamond Contract
  ────────────────────              ────────────────
  
  ┌─────────────────┐               ┌─────────────────┐
  │                 │               │    Diamond      │
  │  All logic in   │               │    (Router)     │
  │  one contract   │               └────────┬────────┘
  │                 │                        │
  │  • Hard to      │               ┌────────┼────────┐
  │    upgrade      │               │        │        │
  │  • Size limits  │               ▼        ▼        ▼
  │  • All or       │         ┌───────┐ ┌───────┐ ┌───────┐
  │    nothing      │         │Facet 1│ │Facet 2│ │Facet 3│
  │                 │         │ Roles │ │Monetary│ │  ...  │
  └─────────────────┘         └───────┘ └───────┘ └───────┘
                                   
                              • Upgrade individual modules
                              • No size limits
                              • Add features over time
                              • Fix bugs without migration
```

---

## Network Deployment

The Stobox infrastructure is deployed on:

| Network | Type | Purpose |
|---------|------|---------|
| **Arbitrum One** | Mainnet | Production deployment |
| **Arbitrum Sepolia** | Testnet | Development and testing |

### Why Arbitrum?

| Benefit | Description |
|---------|-------------|
| **Low Fees** | Significantly reduced transaction costs |
| **Fast Transactions** | Near-instant confirmations |
| **Ethereum Security** | Inherits Ethereum's security guarantees |
| **EVM Compatible** | Full compatibility with Ethereum tools |

---

## Compliance-Native by Design

Compliance within STV3 is not an overlay or external control. It is **embedded into the protocol**:

- Every action requires validation
- Every address must be identity-bound
- Every rule resides within the asset itself

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMPLIANCE-NATIVE ARCHITECTURE                            │
└─────────────────────────────────────────────────────────────────────────────┘

  Traditional Approach              STV3 Approach
  ────────────────────              ─────────────
  
  ┌─────────────────┐               ┌─────────────────┐
  │     TOKEN       │               │   STV3 TOKEN    │
  │   (passive)     │               │   (active)      │
  └────────┬────────┘               └────────┬────────┘
           │                                 │
           ▼                                 │ ← Rules embedded
  ┌─────────────────┐               ┌────────┴────────┐
  │ Transfer Agent  │               │ Validation      │
  │ (manual checks) │               │ Engine          │
  └────────┬────────┘               │ (automatic)     │
           │                        └────────┬────────┘
           ▼                                 │
  ┌─────────────────┐               ┌────────┴────────┐
  │ Compliance Team │               │ DID Layer       │
  │ (manual review) │               │ (identity-bound)│
  └─────────────────┘               └─────────────────┘
  
  Result: Slow, costly,             Result: Instant, automated,
          error-prone                       trustless
```

This design eliminates the need for intermediaries such as transfer agents or manual oversight processes.

**For institutions, this translates into:**
- Reduced operational and legal risk
- Automated adherence to multi-jurisdictional frameworks
- Simplified audits and reporting
- Assets that circulate within permitted investor groups globally without friction

---

## Security Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SECURITY LAYERS                                     │
└─────────────────────────────────────────────────────────────────────────────┘

  Layer 1: NETWORK SECURITY
  ─────────────────────────
  • Arbitrum rollup security
  • Ethereum settlement layer
  
  Layer 2: CONTRACT SECURITY
  ──────────────────────────
  • OpenZeppelin libraries
  • Role-based access control
  • Pausable operations
  
  Layer 3: COMPLIANCE SECURITY
  ────────────────────────────
  • DID verification required
  • Transfer validation rules
  • Investor eligibility checks
  
  Layer 4: INVESTOR PROTECTION
  ────────────────────────────
  • Token reservation mechanism
  • Payment locking until soft cap
  • Refund capabilities
  • Emergency recovery functions
```

---

## Upgradeability and Governance

One of the most powerful features of STV3's architecture is its **upgradeability**. Many protocols fail because their tokens are deployed with fixed logic that becomes outdated as regulations evolve or business models change.

STV3 avoids this by separating logic into facets. Issuers can:
- Add new capabilities
- Refine existing logic
- Adapt to new regulatory environments
- Integrate emerging data sources
- Enhance governance functions

**All without disrupting token supply or investor holdings.** This ensures long-term viability and regulatory resilience.

---

## Web2 and Web3 Interoperability

The STV3 Protocol is designed for **hybrid enterprise environments**.

| Access Layer | Capabilities |
|--------------|--------------|
| **Web2 via Stobox 4** | Onboarding & KYC/KYB, DID issuance, Asset setup & governance, Reporting & dashboards, Lifecycle management |
| **Web3 Direct Access** | EVM wallets, Custody platforms, DeFi protocols, Tokenization marketplaces, Smart contract integrations |

This dual-access model allows businesses to retain traditional workflows while tapping into new distribution channels and technological capabilities.

---

## Designed for Institutional Scale

STV3 was developed with institutional markets in mind. It supports:

| Asset Class | Examples |
|-------------|----------|
| **Securities** | Equity, debt instruments, structured notes |
| **Funds** | Fund units, multi-asset portfolios |
| **Commodities** | Stable-value instruments, production-linked assets |
| **Sustainability** | Carbon markets, ESG-linked assets |
| **Alternatives** | Revenue-share instruments, SPVs |

---

## Strategic Impact for Organizations

Adopting the STV3 Protocol enables organizations to:

| Benefit | Impact |
|---------|--------|
| **Reduce Costs** | Dramatically lower compliance and operational expenses |
| **Expand Globally** | Built-in regulatory alignment across jurisdictions |
| **Automate Operations** | Automated reporting and lifecycle management |
| **Create New Products** | Launch innovative financial products and distribution channels |
| **Build Trust** | Establish stronger investor confidence through transparency |

**STV3 is not simply a token standard. It is a programmable infrastructure for the next generation of real-world assets.**

---

## Key Platform Features

| Feature | Description |
|---------|-------------|
| **One-Click Token Creation** | Deploy complete token infrastructure instantly |
| **Built-in Compliance** | KYC/AML verification for every transaction |
| **Privacy-Preserving KYC** | Hash-based verification protects investor data |
| **Flexible Offerings** | Configure STOs with custom rules and parameters |
| **Investor Protection** | Soft cap enforcement, refund mechanisms |
| **Upgradeable Design** | Add features without token migration |
| **Multi-Wallet Support** | Investors can use multiple wallets with one identity |
| **Role Separation** | Different permissions for different operations |

---

## Getting Started

1. **[Token Lifecycle](./02-token-lifecycle.md)** — Understand the complete journey from creation to trading
2. **[System Components](./components/README.md)** — Deep dive into each component
3. **[Roles and Permissions](./03-roles-and-permissions.md)** — Learn about access control
4. **[Deployed Contracts](./deployed-contracts.md)** — Find contract addresses

---

[← Back to Index](./README.md) | [Next: Token Lifecycle →](./02-token-lifecycle.md)
