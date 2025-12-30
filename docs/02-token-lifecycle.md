# Token Lifecycle

> Complete journey from token creation to investor distribution

---

## Overview

The STV3 token lifecycle represents the complete journey of a tokenized asset — from initial concept to investor distribution through Security Token Offering (STO). Each stage involves specific smart contracts and operations that ensure compliance, security, and proper asset management.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        TOKEN LIFECYCLE OVERVIEW                             │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
  │ STAGE 1 │───►│ STAGE 2 │───►│ STAGE 3 │───►│ STAGE 4 │
  │Prepare  │    │ Create  │    │Configure│    │ Issue   │
  └─────────┘    └─────────┘    └─────────┘    └─────────┘
                                                    │
       ┌────────────────────────────────────────────┘
       │
       ▼
  ┌─────────┐    ┌─────────┐
  │ STAGE 5 │───►│ STAGE 6 │
  │   STO   │    │  Sales  │
  └─────────┘    └─────────┘
```

| Stage | Name | Primary Contract | Key Action |
|-------|------|------------------|------------|
| 1 | Preparation | StoboxDID | Set up issuer identity and compliance |
| 2 | Token Creation | VaultFactory | Deploy token + treasury |
| 3 | Configuration | STV3 Token | Set roles, rules, parameters |
| 4 | Token Issuance | STV3 Token | Mint tokens to treasury |
| 5 | Offering Creation | OfferingRegistry | Configure STO parameters |
| 6 | Activation & Sales | STV3 Token + Registry | Process investor purchases |

---

## Stage 1: Preparation

Before any token can be created, the foundation must be established.

### What Happens

```
┌─────────────────────────────────────────────────────────────────┐
│                    STAGE 1: PREPARATION                          │
└─────────────────────────────────────────────────────────────────┘

  ISSUER                         STOBOX PLATFORM
    │                                  │
    │  1. Legal entity setup           │
    │  2. Asset documentation          │
    │  3. KYC/KYB verification         │
    │─────────────────────────────────►│
    │                                  │
    │                                  │  4. Create Issuer DID
    │                                  │─────────────────────►  StoboxDID
    │                                  │
    │  5. Issuer DID assigned          │
    │◄─────────────────────────────────│
    │                                  │
```

### Key Steps

| Step | Description | Outcome |
|------|-------------|---------|
| **Legal Setup** | Company formation, regulatory compliance | Legal entity ready |
| **Asset Documentation** | Prospectus, terms, asset valuation | Documentation complete |
| **KYC/KYB** | Issuer identity verification | Verified issuer status |
| **DID Creation** | On-chain identity for issuer | Issuer DID on blockchain |

### Contracts Involved

| Contract | Role |
|----------|------|
| **StoboxDID** | Creates and stores issuer identity |

---

## Stage 2: Token Creation

The issuer deploys their security token using the VaultFactory.

### What Happens

```
┌─────────────────────────────────────────────────────────────────┐
│                  STAGE 2: TOKEN CREATION                         │
└─────────────────────────────────────────────────────────────────┘

  ISSUER                    VAULT FACTORY                 RESULT
    │                            │                           │
    │  createToken()             │                           │
    │  - name: "Company Stock"   │                           │
    │  - symbol: "COMP"          │                           │
    │  - supply: 1,000,000       │                           │
    │  - package: 2 (Standard)   │                           │
    │───────────────────────────►│                           │
    │                            │                           │
    │                            │  Deploy STV3 Token        │
    │                            │─────────────────────────► │ 0xToken...
    │                            │                           │
    │                            │  Deploy Treasury          │
    │                            │─────────────────────────► │ 0xTreasury...
    │                            │                           │
    │                            │  Link & Configure         │
    │                            │─────────────────────────► │ Ready
    │                            │                           │
    │  Token + Treasury addresses│                           │
    │◄───────────────────────────│                           │
    │                            │                           │
```

### Token Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| **Name** | Full token name | "Stobox Company Stock" |
| **Symbol** | Trading ticker | "COMP" |
| **Initial Supply** | Total tokens to create | 1,000,000 |
| **Package** | Feature set (Basic/Standard/Advanced) | 2 (Standard) |
| **Decimals** | Token divisibility | 18 |

### What Gets Created

| Contract | Purpose |
|----------|---------|
| **STV3 Token** | The security token itself (Diamond contract) |
| **STV3 Treasury** | Holds tokens and manages payments |

### Contracts Involved

| Contract | Role |
|----------|------|
| **StoboxRWAVaultFactory** | Deploys token and treasury |
| **StoboxProtocolSTV3** | The deployed token contract |
| **STV3Treasury** | The deployed treasury contract |

---

## Stage 3: Token Configuration

After creation, the token must be configured with roles, rules, and parameters.

### What Happens

```
┌─────────────────────────────────────────────────────────────────┐
│                 STAGE 3: TOKEN CONFIGURATION                     │
└─────────────────────────────────────────────────────────────────┘

  ISSUER/ADMIN                   STV3 TOKEN
       │                              │
       │  1. Assign Roles             │
       │  - Financial Operator        │
       │  - Tech Service Role         │
       │  - Recovery Operator         │
       │─────────────────────────────►│
       │                              │
       │  2. Set Validation Rules     │
       │  - HasDIDRule                │
       │─────────────────────────────►│
       │                              │
       │  3. Configure Trust List     │
       │  - Treasury address          │
       │  - Offering Registry         │
       │─────────────────────────────►│
       │                              │
       │  4. Set DID Contract         │
       │─────────────────────────────►│
       │                              │
       │         Configured ✓         │
       │◄─────────────────────────────│
       │                              │
```

### Configuration Steps

| Step | What | Why |
|------|------|-----|
| **Assign Roles** | Grant permissions to wallet addresses | Enable operations by authorized personnel |
| **Set Validation Rules** | Add compliance rules | Ensure regulatory compliance |
| **Configure Trust List** | Whitelist system addresses | Allow treasury and registry operations |
| **Link DID Contract** | Connect to StoboxDID | Enable investor verification |

### Available Roles

| Role | Permissions |
|------|-------------|
| **Owner** | Full control, can grant/revoke all roles |
| **Deployer** | Factory contract, initial setup |
| **Financial Operator** | Mint, burn, treasury operations |
| **Tech Service Role** | System integrations |
| **Recovery Operator** | Emergency operations, forced transfers |

### Contracts Involved

| Contract | Role |
|----------|------|
| **StoboxProtocolSTV3** | Receives configuration |
| **StoboxDID** | Linked for validation |

---

## Stage 4: Token Issuance

Tokens are minted and placed in the treasury, ready for distribution.

### What Happens

```
┌─────────────────────────────────────────────────────────────────┐
│                   STAGE 4: TOKEN ISSUANCE                        │
└─────────────────────────────────────────────────────────────────┘

  FINANCIAL OPERATOR             STV3 TOKEN              TREASURY
         │                           │                       │
         │  issue(treasury, amount)  │                       │
         │──────────────────────────►│                       │
         │                           │                       │
         │                           │  Mint tokens          │
         │                           │  Transfer to treasury │
         │                           │──────────────────────►│
         │                           │                       │
         │                           │                       │ Tokens held
         │                           │                       │ in reserve
         │                           │                       │
         │      Tokens issued ✓      │                       │
         │◄──────────────────────────│                       │
         │                           │                       │

  Token Supply Status:
  ┌──────────────────────────────────────────────────────────┐
  │  Total Supply:    1,000,000 COMP                         │
  │  Treasury:        1,000,000 COMP  (100%)                 │
  │  Circulating:             0 COMP  (0%)                   │
  └──────────────────────────────────────────────────────────┘
```

### Issuance Options

| Method | Description | When to Use |
|--------|-------------|-------------|
| **Issue to Treasury** | Mint all tokens to treasury | Standard STO preparation |
| **Partial Issuance** | Mint portion of supply | Staged token releases |

### Contracts Involved

| Contract | Role |
|----------|------|
| **StoboxProtocolSTV3** | Mints tokens |
| **STV3Treasury** | Receives and holds tokens |

---

## Stage 5: Offering Creation (STO)

Create and configure the Security Token Offering.

### What Happens

```
┌─────────────────────────────────────────────────────────────────┐
│                STAGE 5: OFFERING CREATION                        │
└─────────────────────────────────────────────────────────────────┘

  ISSUER/ADMIN               OFFERING REGISTRY           TREASURY
       │                            │                        │
       │  createOffering()          │                        │
       │  - Token address           │                        │
       │  - Payment token (USDC)    │                        │
       │  - Price per token         │                        │
       │  - Soft cap / Hard cap     │                        │
       │  - Start / End dates       │                        │
       │  - Min/Max investment      │                        │
       │───────────────────────────►│                        │
       │                            │                        │
       │                            │  Reserve tokens        │
       │                            │  from treasury         │
       │                            │───────────────────────►│
       │                            │                        │
       │                            │  Tokens reserved       │
       │                            │◄───────────────────────│
       │                            │                        │
       │  Offering ID: #1           │                        │
       │◄───────────────────────────│                        │
       │                            │                        │
```

### Offering Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| **Token Address** | STV3 token to sell | 0xToken... |
| **Payment Token** | Accepted currency | USDC |
| **Price** | Cost per token | 1.00 USDC |
| **Soft Cap** | Minimum funding goal | 100,000 USDC |
| **Hard Cap** | Maximum funding goal | 1,000,000 USDC |
| **Start Date** | When sales begin | Jan 1, 2025 |
| **End Date** | When sales end | Mar 31, 2025 |
| **Min Investment** | Minimum purchase | 100 USDC |
| **Max Investment** | Maximum per investor | 50,000 USDC |

### Investment Rules

| Rule | Purpose |
|------|---------|
| **HasDIDRule** | Investor must have verified identity |
| **ExcludeUSAandCANRule** | Geographic restrictions |
| **AccreditedInvestorRule** | Accreditation requirements |

### Offering States

```
  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ CREATED  │────►│  ACTIVE  │────►│  PAUSED  │────►│ COMPLETED│
  └──────────┘     └──────────┘     └──────────┘     └──────────┘
       │                │                │                │
       │                │                │                │
       └────────────────┴────────────────┴───────────────►│ CANCELLED
                                                          └──────────┘
```

### Contracts Involved

| Contract | Role |
|----------|------|
| **StoboxRWAOfferingRegistry** | Creates and manages offering |
| **STV3Treasury** | Reserves tokens for sale |

---

## Stage 6: Activation and Sales

The offering goes live and investors can purchase tokens.

### What Happens

```
┌─────────────────────────────────────────────────────────────────┐
│                STAGE 6: ACTIVATION AND SALES                     │
└─────────────────────────────────────────────────────────────────┘

  INVESTOR              STV3 TOKEN          OFFERING REGISTRY      DID
     │                      │                      │                │
     │  1. approve(USDC)    │                      │                │
     │─────────────────────►│                      │                │
     │                      │                      │                │
     │  2. purchaseTokens() │                      │                │
     │─────────────────────►│                      │                │
     │                      │  3. Validate offer   │                │
     │                      │─────────────────────►│                │
     │                      │                      │  4. Check DID  │
     │                      │                      │───────────────►│
     │                      │                      │                │
     │                      │                      │  5. Check rules│
     │                      │                      │◄───────────────│
     │                      │                      │                │
     │                      │  6. Validation OK    │                │
     │                      │◄─────────────────────│                │
     │                      │                      │                │
     │                      │  7. Transfer USDC    │                │
     │                      │     Investor → Treasury               │
     │                      │                      │                │
     │                      │  8. Transfer tokens  │                │
     │  9. Receive tokens   │     Treasury → Investor               │
     │◄─────────────────────│                      │                │
     │                      │                      │                │
```

### Purchase Flow

| Step | Action | Contract |
|------|--------|----------|
| 1 | Investor approves payment token spending | Payment Token (USDC) |
| 2 | Investor calls purchase function | STV3 Token |
| 3 | System validates offering is active | Offering Registry |
| 4 | System checks investor DID exists | StoboxDID |
| 5 | System applies investment rules | Offering Registry |
| 6 | Validation passes | All |
| 7 | Payment transferred to treasury | Treasury |
| 8 | Tokens transferred to investor | STV3 Token |
| 9 | Purchase recorded | Offering Registry |

### Investor Protection Mechanisms

| Mechanism | Description |
|-----------|-------------|
| **Payment Token Locking** | USDC held in treasury until soft cap reached |
| **Soft Cap Protection** | If soft cap not reached, full refund available |
| **Lockup Periods** | Tokens may be locked until offering completes |

### What If Soft Cap Not Reached?

```
┌─────────────────────────────────────────────────────────────────┐
│               SOFT CAP NOT REACHED — REFUND FLOW                 │
└─────────────────────────────────────────────────────────────────┘

  Offering ends with $80,000 raised (Soft Cap was $100,000)

  INVESTOR                   STV3 TOKEN               TREASURY
     │                           │                        │
     │  claimRefund()            │                        │
     │──────────────────────────►│                        │
     │                           │                        │
     │                           │  Return security tokens│
     │                           │───────────────────────►│
     │                           │  Return USDC payment   │
     │  Receive USDC back        │◄───────────────────────│
     │◄──────────────────────────│                        │
     │                           │                        │

  Result: Investor receives full USDC refund
          Tokens return to treasury
```

### Contracts Involved

| Contract | Role |
|----------|------|
| **StoboxProtocolSTV3** | Purchase entry point |
| **StoboxRWAOfferingRegistry** | Validates and records purchase |
| **StoboxDID** | Verifies investor identity |
| **STV3Treasury** | Handles token/payment transfers |

---

## Complete Lifecycle Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       COMPLETE TOKEN LIFECYCLE                               │
└─────────────────────────────────────────────────────────────────────────────┘

                                 PREPARATION
                                     │
            ┌────────────────────────┴────────────────────────┐
            │  • Legal entity setup                          │
            │  • Asset documentation                         │
            │  • Issuer KYC/KYB                              │
            │  • Issuer DID creation                         │
            └────────────────────────┬────────────────────────┘
                                     ▼
                               TOKEN CREATION
                                     │
            ┌────────────────────────┴────────────────────────┐
            │  Factory.createToken()                         │
            │  └── Deploys: STV3 Token + Treasury            │
            └────────────────────────┬────────────────────────┘
                                     ▼
                             CONFIGURATION
                                     │
            ┌────────────────────────┴────────────────────────┐
            │  • Assign roles                                │
            │  • Set validation rules                        │
            │  • Configure trust list                        │
            │  • Link DID contract                           │
            └────────────────────────┬────────────────────────┘
                                     ▼
                              TOKEN ISSUANCE
                                     │
            ┌────────────────────────┴────────────────────────┐
            │  Token.issue(treasury, amount)                 │
            │  └── Tokens minted to treasury                 │
            └────────────────────────┬────────────────────────┘
                                     ▼
                           OFFERING CREATION
                                     │
            ┌────────────────────────┴────────────────────────┐
            │  Registry.createOffering()                     │
            │  └── Sets price, caps, dates, rules            │
            │  └── Reserves tokens from treasury             │
            └────────────────────────┬────────────────────────┘
                                     ▼
                          ACTIVATION & SALES
                                     │
            ┌────────────────────────┴────────────────────────┐
            │  Investors purchase tokens                     │
            │  └── DID verification                          │
            │  └── Rule validation                           │
            │  └── Payment → Treasury                        │
            │  └── Tokens → Investor                         │
            └─────────────────────────────────────────────────┘
```

---

## Lifecycle Events and Notifications

| Stage | Event | Contract | Trigger |
|-------|-------|----------|---------|
| 2 | `SecurityTokenCreated` | VaultFactory | New token deployed |
| 2 | `TreasuryInit` | Treasury | Treasury initialized |
| 4 | `Issued` | STV3 Token | Tokens minted |
| 5 | `NewOfferingCreated` | OfferingRegistry | STO configured |
| 6 | `PurchaseReceipt` | OfferingRegistry | Purchase recorded |
| 6 | `STV3TokenPurchased` | STV3 Token | Investor buys tokens |
| 6 | `STV3TokenPurchaseRefunded` | STV3 Token | Investor refund processed |
| 6 | `RefundedPurchaseReceipt` | OfferingRegistry | Refund recorded |

---

## Related Documentation

- [Architecture Overview](./01-architecture-overview.md) — System design
- [VaultFactory](./components/vault-factory.md) — Token creation details
- [STV3 Protocol](./components/protocol-stv3.md) — Token functionality
- [Offering Registry](./components/offering-registry.md) — STO management
- [StoboxDID](./components/stobox-did.md) — Identity verification

---

[← Back to Index](./README.md) | [Previous: Architecture](./01-architecture-overview.md) | [Next: System Components →](./components/README.md)
