# StoboxRWAOfferingRegistry

> The offering management system — where security token sales are configured, launched, and tracked

---

## Overview

**StoboxRWAOfferingRegistry** is the smart contract that manages Security Token Offerings (STOs). Think of it as the "sales platform" for tokenized assets — it handles everything from setting up an offering to processing investor purchases and tracking the entire fundraising process.

When a company wants to sell their security tokens to investors, they register an offering in this Registry. The Registry then manages the entire sale: who can invest, how much, when, and under what conditions.

---

## Purpose

| Function | Description |
|----------|-------------|
| **Offering Management** | Create, configure, and control token sales |
| **Investor Purchases** | Process and record all investments |
| **Compliance Enforcement** | Apply investment rules automatically |
| **Fundraising Tracking** | Monitor progress toward funding goals |
| **Lockup Management** | Configure token restriction periods |

---

## What is a Security Token Offering (STO)?

An STO is a regulated fundraising method where a company sells security tokens to investors. Unlike ICOs, STOs are compliant with securities regulations.

```
┌─────────────────────────────────────────────────────────────────┐
│                    SECURITY TOKEN OFFERING                       │
└─────────────────────────────────────────────────────────────────┘

     Company                    STO                      Investors
        │                        │                           │
        │   1. Create Token      │                           │
        │──────────────────────►│                           │
        │                        │                           │
        │   2. Register Offering │                           │
        │──────────────────────►│                           │
        │                        │                           │
        │   3. Set Rules         │                           │
        │──────────────────────►│                           │
        │                        │                           │
        │   4. Activate          │                           │
        │──────────────────────►│                           │
        │                        │   5. Purchase Tokens      │
        │                        │◄──────────────────────────│
        │                        │                           │
        │                        │   6. Receive Tokens       │
        │                        │──────────────────────────►│
        │                        │                           │
        │   7. Receive Funds     │                           │
        │◄──────────────────────│                           │
        │                        │                           │
```

---

## Offering Parameters

When creating an offering, the following parameters are configured:

| Parameter | Description | Example |
|-----------|-------------|---------|
| **STO ID** | Unique identifier for the offering | "ACME-STO-2024" |
| **Name** | Offering name | "Acme Real Estate Series A" |
| **Description** | Details about the offering | "Investment in Manhattan property" |
| **Token Address** | The security token being sold | 0x1234...5678 |
| **Start Date** | When the offering opens | January 1, 2025 |
| **End Date** | When the offering closes | March 31, 2025 |
| **Token Price** | Price per token in USD | $10.00 |
| **Minimum Investment** | Smallest allowed investment | $1,000 |
| **Offering Amount** | Total tokens available for sale | 100,000 tokens |
| **Payment Tokens** | Accepted currencies | USDC |
| **Rules** | Compliance rules to apply | Exclude USA and Canada Rule |

---

## Offering Lifecycle

An offering moves through different statuses during its lifetime:

```
┌─────────────────────────────────────────────────────────────────┐
│                    OFFERING LIFECYCLE                            │
└─────────────────────────────────────────────────────────────────┘

                         ┌───────────┐
                         │ SCHEDULED │ (Created, waiting to start)
                         └─────┬─────┘
                               │ activate
                               ▼
                         ┌───────────┐
              ┌─────────►│ ACTIVATED │◄─────────┐
              │          └─────┬─────┘          │
              │                │                │
         unpause │ pause       │           unsuspend │ suspend 
              │                │                │
              │                ▼                │
        ┌─────┴─────┐    ┌───────────┐    ┌────┴──────┐
        │  PAUSED   │    │           │    │ SUSPENDED │
        └───────────┘    │           │    └───────────┘
                         │           │
              ┌──────────┴───────────┴──────────┐
              │                │                │
              ▼                ▼                ▼
        ┌───────────┐    ┌───────────┐    ┌───────────┐
        │ COMPLETED │    │ CANCELED  │    │TERMINATED │
        └───────────┘    └───────────┘    └───────────┘
         (Success)       (Before start)   (Force stop)
```

### Status Descriptions

| Status | Description | Purchases Allowed |
|--------|-------------|-------------------|
| **SCHEDULED** | Offering created, waiting for start date | ❌ No |
| **ACTIVATED** | Offering is live and accepting investments | ✅ Yes |
| **PAUSED** | Temporarily stopped (can be resumed) | ❌ No |
| **SUSPENDED** | Suspended for review (can be resumed) | ❌ No |
| **COMPLETED** | Successfully reached goals or end date | ❌ No |
| **CANCELED** | Canceled before activation | ❌ No |
| **TERMINATED** | Forcefully ended by administrator | ❌ No |

---

## Investment Rules

The Registry enforces compliance through configurable **rules**. Each rule checks specific conditions before allowing a purchase.

### Available Rules

| Rule | Description |
|------|-------------|
| **ExcludeUSAandCANRule** | Blocks investors from USA and Canada |
| **AccreditedUSMinInvestmentRule** | Requires US investors to be accredited with minimum investment |
| **ExcludeNonAccreditedUSMinInvestmentRule** | Excludes non-accredited US investors, sets minimum for others |

### How Rules Work

```
┌─────────────────────────────────────────────────────────────────┐
│                    RULE VALIDATION FLOW                          │
└─────────────────────────────────────────────────────────────────┘

  Investor Purchase Request
           │
           ▼
    ┌─────────────┐
    │  Read DID   │ (Get investor's identity attributes)
    │  Attributes │
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐     ┌─────────────────────────────────┐
    │   Rule 1    │────►│ Check: Is investor from USA/CAN? │
    └──────┬──────┘     └─────────────────────────────────┘
           │                         │
           │ Pass                    │ Fail → ❌ Rejected
           ▼                         
    ┌─────────────┐     ┌─────────────────────────────────┐
    │   Rule 2    │────►│ Check: Meets minimum investment? │
    └──────┬──────┘     └─────────────────────────────────┘
           │                         │
           │ Pass                    │ Fail → ❌ Rejected
           ▼
    ✅ Purchase Approved
```

---

## Purchase Process

When an investor wants to buy tokens:

### Step-by-Step Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    PURCHASE PROCESS                              │
└─────────────────────────────────────────────────────────────────┘

  1. INVESTOR INITIATES PURCHASE
     └─► Specifies: amount of tokens, payment token
  
  2. VALIDATION CHECKS
     ├─► Is offering active?
     ├─► Does investor have valid DID?
     ├─► Do all rules pass?
     ├─► Is minimum investment met?
     └─► Are enough tokens available?
  
  3. PRICE CALCULATION
     └─► Token amount × Price in USD = Payment required
  
  4. PAYMENT APPROVAL (prerequisite)
     └─► Investor must approve the contract to spend their payment tokens
         (standard ERC-20 approve mechanism)
  
  5. PAYMENT TRANSFER
     └─► Payment tokens transferred from investor to Treasury
  
  6. TOKEN DISTRIBUTION
     └─► Security tokens transferred from Treasury to investor
  
  7. LOCKUP APPLIED (if configured)
     └─► Tokens locked until specified date
  
  8. PURCHASE RECORDED
     └─► Transaction details stored in Registry
```

### Purchase Receipt

Each purchase generates a receipt with:

| Field | Description |
|-------|-------------|
| **STO ID** | Which offering |
| **Investor Address** | Who purchased |
| **Security Token Amount** | How many tokens |
| **Payment Token** | What currency used |
| **Payment Amount** | How much paid |
| **Price in USD** | Price at time of purchase |
| **Purchase Date** | When purchased |
| **Lockup Date** | When tokens unlock |
| **Note** | Additional information |

---

## Token Reservation Mechanism

When an offering is created, the system **reserves security tokens** on the Treasury to guarantee availability for investors.

### How Reservation Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    TOKEN RESERVATION                             │
└─────────────────────────────────────────────────────────────────┘

  OFFERING CREATION
  ─────────────────
  
  1. Offering Amount: 100,000 tokens
     └─► System checks: Does Treasury have 100,000 tokens available?
  
  2. If YES → Tokens are RESERVED
     └─► Reserved tokens cannot be used for other offerings
     └─► Reserved tokens cannot be withdrawn from Treasury
  
  3. Offering goes live
     └─► Each purchase reduces reserved amount
     └─► Reserved Amount = Offering Amount - Sold Amount
```

### Reservation Tracking

| Metric | Description |
|--------|-------------|
| **Offering Amount** | Total tokens allocated for this offering |
| **Reserved on Treasury** | Tokens still available for purchase |
| **Sold Amount** | Tokens already purchased by investors |
| **Aggregated Reserved** | Total reserved across all active offerings for a token |

This mechanism ensures that:
- ✅ Tokens are always available when investors purchase
- ✅ Multiple offerings cannot oversell the same tokens
- ✅ Treasury balance accurately reflects available vs. reserved tokens

---

## Soft Cap and Hard Cap

Offerings can have funding goals:

| Term | Description |
|------|-------------|
| **Soft Cap** | Minimum funding needed for the offering to succeed |
| **Hard Cap** | Maximum funding the offering will accept (= Offering Amount) |

### Investor Protection: Payment Token Locking

**Important:** Until the soft cap is reached, all payment tokens (e.g., USDC) received from investors are **locked on the Treasury** and cannot be withdrawn by the company.

```
┌─────────────────────────────────────────────────────────────────┐
│                PAYMENT TOKEN PROTECTION                          │
└─────────────────────────────────────────────────────────────────┘

  DURING OFFERING (Soft Cap NOT yet reached)
  ──────────────────────────────────────────
  
  Treasury Status:
  ┌─────────────────────────────────────────┐
  │  Security Tokens: 100,000 (reserved)    │
  │  Payment Tokens:   25,000 USDC (LOCKED) │ ← Cannot withdraw!
  │                                         │
  │  Soft Cap: 50,000 tokens                │
  │  Sold:     25,000 tokens                │
  │  Status:   Soft cap NOT reached         │
  └─────────────────────────────────────────┘
  
  Company CANNOT access the 25,000 USDC until soft cap is met.
  This protects investors in case the offering fails.
```

### Soft Cap Scenarios

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOFT CAP SCENARIOS                            │
└─────────────────────────────────────────────────────────────────┘

  Scenario A: Soft Cap Reached ✅
  ───────────────────────────────
  Offering Amount: 100,000 tokens
  Soft Cap:         50,000 tokens
  Sold:             75,000 tokens  ✅ Above soft cap
  
  Result: Offering SUCCESSFUL
          → Payment tokens UNLOCKED for company
          → Company can withdraw funds from Treasury
          → Investors keep their security tokens

  Scenario B: Soft Cap NOT Reached ❌
  ────────────────────────────────────
  Offering Amount: 100,000 tokens
  Soft Cap:         50,000 tokens
  Sold:             30,000 tokens  ❌ Below soft cap
  
  Result: Offering FAILED → REFUND PROCESS
          → Payment tokens returned to investors
          → Security tokens returned to Treasury (reserved amount restored)
          → No funds lost by investors
```

### Refund Process

If soft cap is not reached by the offering end date:

```
┌─────────────────────────────────────────────────────────────────┐
│                    REFUND FLOW                                   │
└─────────────────────────────────────────────────────────────────┘

  1. Offering ends with soft cap NOT reached
     └─► Status changes to FAILED_SOFTCAP
  
  2. Refund process initiated
     └─► System iterates through all purchases
  
  3. For each purchase:
     ├─► Security tokens returned from investor to Treasury
     ├─► Payment tokens returned from Treasury to investor
     └─► Lockup cleared (if any)
  
  4. Refund completed
     └─► Reserved amounts restored
     └─► Offering marked as REFUNDED
```

---

## Lockup Periods

Tokens purchased during an offering can be locked for a specified period:

### Lockup Reference Points

| Reference Point | Description |
|-----------------|-------------|
| **STO End Date** | Lockup period starts when offering ends |
| **Purchase Date** | Lockup period starts from each individual purchase |

### Example

```
Lockup Configuration:
- Reference Point: STO End Date
- Lockup Period: 180 days

STO Timeline:
├── Jan 1: STO Starts
├── Jan 15: Investor A purchases → Tokens locked
├── Feb 1: Investor B purchases → Tokens locked
├── Mar 31: STO Ends
└── Sep 27: All tokens unlock (180 days after Mar 31)
```

---

## Distribution Type

The Registry supports different ways to distribute tokens:

| Type | Description |
|------|-------------|
| **Pre-Mint** | Tokens are minted to Treasury before offering starts. Purchases transfer from Treasury. |
| **Mint-on-Purchase** | Tokens are minted directly to investor at time of purchase. (Future feature) |

Currently, **Pre-Mint** is the standard distribution method.

---

## Key Features Summary

| Feature | Benefit |
|---------|---------|
| **Flexible Configuration** | Customize every aspect of your offering |
| **Automatic Compliance** | Rules enforced on every purchase |
| **Real-time Tracking** | Monitor sales progress instantly |
| **Multiple Payment Options** | Accept various stablecoins |
| **Lockup Management** | Configure vesting schedules easily |
| **Soft Cap Protection** | Automatic handling if minimums not met |
| **Complete Audit Trail** | Every purchase recorded on-chain |

---

## Integration with Other Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    SYSTEM INTEGRATION                            │
└─────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────┐
                    │  Offering Registry  │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
    ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
    │   STV3      │     │  StoboxDID  │     │   Rules     │
    │   Token     │     │  (Identity) │     │ (Compliance)│
    └─────────────┘     └─────────────┘     └─────────────┘
           │                   │                   │
           │                   │                   │
           ▼                   ▼                   ▼
    Token transfers      Verify investor     Check eligibility
    Treasury access      Get attributes      Apply restrictions
```

---

## Related Documentation

- [StoboxProtocolSTV3](./protocol-stv3.md) — The token being sold
- [StoboxDID](./stobox-did.md) — Investor identity verification
- [Compliance and Security](../04-compliance-and-security.md) — Detailed compliance rules

---

[← Back to Components](./README.md) | [Previous: Protocol STV3](./protocol-stv3.md) | [Next: Stobox DID →](./stobox-did.md)
