# StoboxDID

> Decentralized identity system — the foundation of compliant investor verification

---

## Overview

**StoboxDID** (Stobox Decentralized Identity) is the identity layer of the Stobox platform. It provides a blockchain-based system for verifying investor identities and storing compliance-related attributes.

In the world of security tokens, knowing your investor is not optional — it's a regulatory requirement. StoboxDID solves this by creating a decentralized, portable identity that travels with the investor across all Stobox-powered offerings.

---

## Why Decentralized Identity?

Traditional identity verification has problems:

| Traditional KYC | StoboxDID |
|-----------------|-----------|
| Repeated verification for each platform | Verify once, use everywhere |
| Data stored in centralized databases | Data controlled on blockchain |
| Paper-based, slow processes | Instant on-chain verification |
| No investor control | Investor manages their identity |
| Limited interoperability | Works across all Stobox tokens |

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    STOBOX DID SYSTEM                             │
└─────────────────────────────────────────────────────────────────┘

  1. IDENTITY CREATION
     ┌─────────────────────────────────────────────────────────┐
     │  Investor initiates KYC/AML verification process        │
     │  After verification, DID is created on blockchain       │
     └─────────────────────────────────────────────────────────┘
                              │
                              ▼
  2. ATTRIBUTE ASSIGNMENT
     ┌─────────────────────────────────────────────────────────┐
     │  Attributes added: Country, Accreditation status, etc.  │
     │  Each attribute has validity period                     │
     └─────────────────────────────────────────────────────────┘
                              │
                              ▼
  3. WALLET LINKING
     ┌─────────────────────────────────────────────────────────┐
     │  Investor's wallet addresses linked to their DID        │
     │  Multiple wallets can share one identity                │
     └─────────────────────────────────────────────────────────┘
                              │
                              ▼
  4. ONGOING VERIFICATION
     ┌─────────────────────────────────────────────────────────┐
     │  Every token transfer checks DID automatically          │
     │  Every STO purchase validates investor eligibility      │
     └─────────────────────────────────────────────────────────┘
```

---

## DID Structure

Each Decentralized Identity contains:

| Component | Description |
|-----------|-------------|
| **uDID** | Unique identifier (e.g., "INV-2024-00123") |
| **Valid To** | Expiration date of the identity |
| **Blocked Status** | Whether the DID is currently blocked |
| **Attributes** | Key-value pairs with investor information |
| **Linked Addresses** | Wallet addresses associated with this identity |
| **Last Updated** | When the DID was last modified |

---

## Attributes

Attributes store verified information about the investor. Each attribute has:
- **Name** — What the attribute represents
- **Value** — The verified data (stored as encrypted bytes)
- **Value Type** — Describes how to interpret the data
- **Valid To** — When the attribute expires
- **Created/Updated** — Timestamps
- **Last Updated By** — Address that made the last change

### Data Storage Format

Attribute values are stored **as bytes** on the blockchain, which provides several benefits:

| Benefit | Description |
|---------|-------------|
| **Encryption Support** | Data can be stored in encrypted form |
| **Flexibility** | Any data type can be encoded (strings, numbers, complex structures) |
| **Privacy** | Raw data is not human-readable on blockchain explorers |
| **Standardization** | Consistent storage format across all attributes |

```
┌─────────────────────────────────────────────────────────────────┐
│                    ATTRIBUTE STORAGE                             │
└─────────────────────────────────────────────────────────────────┘

  Attribute Example: COUNTRY
  
  ┌─────────────────────────────────────────┐
  │  attributeName:  "COUNTRY"              │
  │  value:          0x44455500... (bytes)  │ ← Encoded/encrypted data
  │  valueType:      "string"               │ ← How to interpret
  │  validTo:        1735689600             │ ← Expiration timestamp
  │  createdAt:      1704067200             │
  │  updatedAt:      1704067200             │
  │  lastUpdatedBy:  0xABC...123            │
  └─────────────────────────────────────────┘
  
  The "valueType" field tells systems how to decode the bytes:
  - "string"  → decode as text
  - "bytes"   → raw bytes / hash
  - "address" → Ethereum address
  - "uint256" → unsigned integer
  - "int256"  → signed integer
  - "bool"    → true / false
```

This approach ensures that sensitive investor data remains protected while still being verifiable on-chain.

### Privacy-Preserving Verification

StoboxDID uses advanced cryptographic techniques inspired by **Zero Knowledge Proof** principles. This allows the system to verify investor eligibility **without revealing their actual personal data**.

#### How It Works: Hash Commitment Scheme

```
┌─────────────────────────────────────────────────────────────────┐
│              PRIVACY-PRESERVING VERIFICATION                     │
└─────────────────────────────────────────────────────────────────┘

  STORING DATA (during KYC)
  ─────────────────────────
  
  Instead of storing:  "USA"  ← Readable by anyone
  
  System stores:       hash(investorAddress + "USA")  
                       ↓
                       0x7f83b1657ff1fc53b92dc18148a1d65d...
                       ↑
                       Unreadable, unique to this investor

  VERIFICATION (during investment)
  ─────────────────────────────────
  
  Question: "Is this investor from USA?"
  
  ┌────────────────────────────────────────────────────────┐
  │  1. Take investor's wallet address                     │
  │  2. Calculate: hash(address + "USA")                   │
  │  3. Compare with stored hash                           │
  │                                                        │
  │  Match?     → Investor IS from USA     → ❌ Rejected   │
  │  No match?  → Investor is NOT from USA → ✅ Proceed    │
  └────────────────────────────────────────────────────────┘
  
  The actual country is NEVER revealed — only yes/no answer!
```

#### Cryptographic Properties

| Property | Description | Benefit |
|----------|-------------|---------|
| **Hiding** | Hash conceals the actual data | Country code cannot be read from blockchain |
| **Binding** | Hash is tied to specific address | Cannot be reused or transferred |
| **Salted** | Investor address acts as unique "salt" | Same country = different hash for different investors |
| **One-way** | Cannot reverse hash to get original data | Even with blockchain access, data stays private |

#### Why This Matters

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRIVACY COMPARISON                            │
└─────────────────────────────────────────────────────────────────┘

  Traditional System               StoboxDID
  ──────────────────               ─────────
  
  Stored: "John Smith, USA"        Stored: 0x7f83b165...
  
  Anyone can see:                  Anyone can see:
  ✗ Full name                      ✓ Only cryptographic hash
  ✗ Country                        ✓ Nothing readable
  ✗ Personal details               ✓ Privacy preserved
  
  Verification:                    Verification:
  "Show me the data"               "Does hash match USA?"
  ↓                                ↓
  Reveals everything               Reveals only yes/no
```

This approach provides **regulatory compliance** (KYC verification works) while maintaining **investor privacy** (personal data stays confidential).

### Common Attributes

| Attribute | Description | Value Type | Example |
|-----------|-------------|------------|---------|
| **COUNTRY** | Investor's country of residence | bytes (hashed) | `0x7a9f...` (hashed country code) |
| **TYPE** | Type of investor | string | "INDIVIDUAL" / "INSTITUTIONAL" |
| **ACCREDITED** | Accredited investor status | bool | 0 (false) / 1 (true) |

### How Attributes Are Used

```
┌─────────────────────────────────────────────────────────────────┐
│                    ATTRIBUTE CHECK EXAMPLE                       │
└─────────────────────────────────────────────────────────────────┘

  Investor wants to participate in STO with rule:
  "Exclude USA and Canada Rule"
  
  System checks:
  ┌────────────────────────────────────────┐
  │  1. Get investor's wallet address      │
  │  2. Find linked DID                    │
  │  3. Read COUNTRY attribute             │
  │  4. Check: Is COUNTRY = "USA" or "CAN"?│
  └────────────────────────────────────────┘
            │                    │
            ▼                    ▼
      COUNTRY = "DEU"      COUNTRY = "USA"
            │                    │
            ▼                    ▼
      ✅ Allowed           ❌ Rejected
```

---

## Linked Addresses

One investor can have multiple wallet addresses linked to their single DID:

```
┌─────────────────────────────────────────────────────────────────┐
│                    LINKED ADDRESSES                              │
└─────────────────────────────────────────────────────────────────┘

                        ┌─────────────────┐
                        │   Investor DID  │
                        │  "INV-2024-001" │
                        └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
        ┌───────────┐      ┌───────────┐      ┌───────────┐
        │  Wallet 1 │      │  Wallet 2 │      │  Wallet 3 │
        │  0xAAA... │      │  0xBBB... │      │  0xCCC... │
        │ (Primary) │      │ (Trading) │      │ (Cold)    │
        └───────────┘      └───────────┘      └───────────┘
              │                  │                  │
              └──────────────────┴──────────────────┘
                                 │
                                 ▼
                    All wallets share the same:
                    - KYC verification
                    - Country attribute
                    - Accreditation status
                    - Investment eligibility
```

### Benefits of Linked Addresses

| Benefit | Description |
|---------|-------------|
| **Flexibility** | Use different wallets for different purposes |
| **Single KYC** | Verify once, use all wallets |
| **Portfolio Management** | Separate holdings across wallets |
| **Security** | Keep cold storage wallet linked |
| **Institutional Support** | Organizations can link multiple operational wallets |

### Limits

- Maximum **10 linked addresses** per DID (configurable)
- Each address can only be linked to **one DID**
- Addresses can be **deactivated** without unlinking

---

## DID Lifecycle

### States

| State | Description | Can Transact |
|-------|-------------|--------------|
| **Active** | Normal state, fully operational | ✅ Yes |
| **Expired** | Valid To date has passed | ❌ No |
| **Blocked** | Manually blocked by compliance | ❌ No |

### Address States

Individual linked addresses can also be:

| State | Description |
|-------|-------------|
| **Active** | Address can be used normally |
| **Deactivated** | Temporarily disabled (can be reactivated) |

---

## Access Control

Different roles can perform different actions:

| Role | Permissions |
|------|-------------|
| **Default Admin** | Full control, manage other roles |
| **Writer Role** | Create DIDs, add attributes, manage addresses |
| **Attribute Reader Role** | Read DID attributes (for compliance checks) |

### External Readers

Token contracts and offering registries can be granted **temporary read access** to verify investor attributes:

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXTERNAL READER ACCESS                        │
└─────────────────────────────────────────────────────────────────┘

  Token Contract                         StoboxDID
       │                                     │
       │  "Can I read attributes for         │
       │   wallet 0xAAA...?"                 │
       │─────────────────────────────────────►
       │                                     │
       │      Check: Is token contract       │
       │      an authorized reader?          │
       │                                     │
       │◄────────────────────────────────────│
       │  "Yes, here are the attributes"     │
       │                                     │
```

---

## Integration with Stobox Ecosystem

StoboxDID is the identity foundation for the entire platform:

```
┌─────────────────────────────────────────────────────────────────┐
│                    DID INTEGRATION                               │
└─────────────────────────────────────────────────────────────────┘

                         ┌─────────────┐
                         │  StoboxDID  │
                         │  (Identity) │
                         └──────┬──────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
  ┌───────────────┐     ┌───────────────┐     ┌───────────────┐
  │ HasDIDRule    │     │ Offering      │     │ Investment    │
  │ (Token        │     │ Registry      │     │ Rules         │
  │  Transfers)   │     │ (Purchases)   │     │ (Compliance)  │
  └───────────────┘     └───────────────┘     └───────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
  "Does sender &         "Is investor         "What country?
   receiver have          verified?"           Accredited?"
   valid DID?"
```

### Use Cases

| Component | How DID is Used |
|-----------|-----------------|
| **Token Transfers** | HasDIDRule checks both sender and receiver have valid DID |
| **STO Purchases** | Verify investor identity before allowing investment |
| **Investment Rules** | Read attributes (country, accreditation) for compliance |

---

## Privacy Considerations

| Aspect | Approach |
|--------|----------|
| **On-chain Data** | Only essential identifiers stored on blockchain |
| **Sensitive Data** | Detailed personal info stored off-chain |
| **Access Control** | Only authorized contracts can read attributes |
| **Audit Trail** | All access logged via blockchain events |
| **Right to Erasure** | DID can be blocked, addresses unlinked |

---

## Key Benefits

| Benefit | Description |
|---------|-------------|
| **Verify Once** | Complete KYC once, invest in multiple offerings |
| **Portability** | Identity works across all Stobox-powered tokens |
| **Real-time Compliance** | Automatic verification on every transaction |
| **Regulatory Ready** | Meets KYC/AML requirements for security tokens |
| **Flexible Management** | Add/remove wallets, update attributes as needed |
| **Institutional Support** | Works for individuals and organizations |

---

## Related Documentation

- [StoboxProtocolSTV3](./protocol-stv3.md) — Token that uses DID for validation
- [StoboxRWAOfferingRegistry](./offering-registry.md) — STO system that verifies investors
- [Compliance and Security](../04-compliance-and-security.md) — How DID enables compliance

---

[← Back to Components](./README.md) | [Previous: Offering Registry](./offering-registry.md)
