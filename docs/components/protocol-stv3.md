# StoboxProtocolSTV3

> The security token — a compliant, upgradeable digital representation of real-world assets

---

## Overview

**StoboxProtocolSTV3** is the security token itself — the digital asset that represents ownership in a real-world asset. When you tokenize real estate, company equity, or any other asset through Stobox, this is what gets created.

Unlike simple cryptocurrency tokens, StoboxProtocolSTV3 is designed specifically for **regulated securities**. It includes built-in compliance, investor verification, transfer restrictions, and administrative controls required by securities regulations worldwide.

---

## Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Standard** | ERC-20 compatible (works with wallets, exchanges, DeFi) |
| **RWA Interface** | [ERC-7943 (uRWA)](https://eips.ethereum.org/EIPS/eip-7943) compliant for universal RWA interoperability |
| **Architecture** | Diamond Pattern (EIP-2535) — modular and upgradeable |
| **Compliance** | Built-in validation rules for every transfer |
| **Access Control** | Role-based permissions for different operations |
| **Flexibility** | Can be customized and upgraded without redeployment |

---

## What Makes It Different from Regular Tokens?

```
┌─────────────────────────────────────────────────────────────────┐
│              REGULAR TOKEN vs SECURITY TOKEN                     │
├─────────────────────────────┬───────────────────────────────────┤
│       Regular ERC-20        │      StoboxProtocolSTV3           │
├─────────────────────────────┼───────────────────────────────────┤
│ Anyone can transfer         │ Only verified investors           │
│ No identity checks          │ DID verification required         │
│ No transfer limits          │ Configurable restrictions         │
│ Fixed functionality         │ Upgradeable modules               │
│ No compliance built-in      │ Compliance by design              │
│ Single admin (owner)        │ Multiple specialized roles        │
│ No recovery options         │ Emergency recovery functions      │
└─────────────────────────────┴───────────────────────────────────┘
```

---

## Modular Architecture (Diamond Pattern)

StoboxProtocolSTV3 uses the **Diamond Pattern** (EIP-2535), which means the token is built from separate modules called "facets." Think of it like a smartphone with apps — the core device stays the same, but you can add, update, or remove apps.

### Benefits of Modular Design

| Benefit | Description |
|---------|-------------|
| **Upgradeable** | Add new features without creating a new token |
| **Gas Efficient** | Only deploy what you need |
| **Maintainable** | Fix issues in specific modules without affecting others |
| **Customizable** | Choose which features your token needs |
| **Future-Proof** | Adapt to new regulations or requirements |

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    StoboxProtocolSTV3                           │
│                   (Diamond Contract)                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │    Roles     │  │   Monetary   │  │  Validation  │           │
│  │    Facet     │  │    Facet     │  │    Facet     │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │   Purchase   │  │    Lockup    │  │  Emergency   │           │
│  │    Facet     │  │    Facet     │  │    Facet     │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Token Modules (Facets)

Each module handles a specific set of functions:

### 1. RolesFacet — Access Control

Manages who can do what with the token.

| Role | Permissions |
|------|-------------|
| **Owner** | Full administrative control, can assign other roles |
| **Deployer** | Technical administration, configuration |
| **Financial Operator** | Issue tokens, manage treasury, transfers |
| **Recovery Operator** | Emergency operations, forced transfers |

### 2. MonetaryFacet — Token Supply Management

Controls the creation and destruction of tokens.

| Function | Description |
|----------|-------------|
| **Issue** | Create new tokens (mint to Treasury) |
| **Redeem** | Destroy tokens (burn from Treasury) |
| **Transfer from Treasury** | Distribute tokens to investors |
| **Set Max Issuance** | Define the maximum tokens that can be issued |

### 3. ValidationFacet — Compliance Rules

Ensures every transfer meets regulatory requirements.

| Function | Description |
|----------|-------------|
| **Trust/Distrust** | Manage whitelist of trusted addresses |
| **Link Rules** | Attach compliance rules (e.g., require DID) |
| **Pause Token** | Temporarily stop all transfers |
| **Validation Check** | Automatic verification before every transfer |

### 4. PurchaseFacet — Investor Purchases

Handles the purchase of tokens during offerings.

| Function | Description |
|----------|-------------|
| **Purchase** | Buy tokens with payment tokens (e.g., USDC) |
| **Preview Purchase** | Calculate costs before buying |
| **Set Offering Registry** | Connect to STO management system |

### 5. LockupFacet — Token Restrictions

Manages time-based restrictions on token transfers.

| Function | Description |
|----------|-------------|
| **Lock Tokens** | Restrict tokens until a specific date |
| **Clear Lockups** | Remove lockup restrictions |
| **Check Balance** | View locked vs. available balance |

### 6. EmergencyFacet — Recovery Operations

Provides tools for exceptional situations (regulatory requirements, lost access, etc.).

| Function | Description |
|----------|-------------|
| **Forced Transfer** | Move tokens between addresses (compliance) |
| **Forced Mint** | Create tokens in emergency situations |
| **Forced Burn** | Destroy tokens when required |
| **Pause Protocol** | Stop all protocol operations |

---

## Core Token Functions

These are standard ERC-20 functions built directly into the token and cannot be removed. They ensure compatibility with wallets, exchanges, and other blockchain applications:

| Function | Description |
|----------|-------------|
| `name()` | Returns token name (e.g., "Acme Real Estate Token") |
| `symbol()` | Returns ticker symbol (e.g., "ACME") |
| `decimals()` | Returns decimal places (typically 18) |
| `totalSupply()` | Returns total tokens in circulation |
| `balanceOf(address)` | Returns token balance of an address |
| `maxSupply()` | Returns maximum possible supply |
| `transfer()` | Transfer tokens to another address |
| `approve()` | Allow another address to spend your tokens |
| `transferFrom()` | Transfer tokens on behalf of another address |

---

## Treasury Integration

Every StoboxProtocolSTV3 token works with a dedicated **Treasury** contract:

```
┌─────────────────────┐         ┌─────────────────────┐
│                     │         │                     │
│  StoboxProtocolSTV3 │◄───────►│      Treasury       │
│     (Token)         │         │      (Vault)        │
│                     │         │                     │
└─────────────────────┘         └─────────────────────┘
        │                                │
        │                                │
        ▼                                ▼
   Token Logic                    Holds Assets:
   - Transfers                    - Security Tokens
   - Balances                     - Payment Tokens
   - Compliance                   - Payment Tokens (e.g., USDC)
```

The Treasury:
- Receives newly issued tokens
- Stores investor payments during offerings
- Provides secure fund management
- Is controlled only by the token contract

---

## Transfer Validation Flow

Every token transfer goes through a validation process:

```
┌──────────────────────────────────────────────────────────────┐
│                    TRANSFER VALIDATION                        │
└──────────────────────────────────────────────────────────────┘

  Sender                                              Receiver
    │                                                     │
    └──────────────► Transfer Request ◄───────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ Token Paused? │──── Yes ──► ❌ Rejected
                    └───────┬───────┘
                            │ No
                            ▼
                    ┌───────────────┐
                    │ Sender has    │──── No ───► ❌ Rejected
                    │ enough balance│
                    └───────┬───────┘
                            │ Yes
                            ▼
                    ┌───────────────┐
                    │ Sender locked │──── Yes ──► ❌ Rejected
                    │ balance check │
                    └───────┬───────┘
                            │ No
                            ▼
                    ┌───────────────┐
                    │  Validation   │──── Fail ─► ❌ Rejected
                    │    Rules      │
                    │ (DID, Trust)  │
                    └───────┬───────┘
                            │ Pass
                            ▼
                      ✅ Transfer Executed
```

---

## Token Lifecycle States

A token can exist in different states:

| State | Description | Transfers Allowed |
|-------|-------------|-------------------|
| **Active** | Normal operation | ✅ Yes (with validation) |
| **Protocol Paused** | Emergency stop | ❌ No |

---

## ERC-7943 (uRWA) Compliance

StoboxProtocolSTV3 implements the **Universal Real World Asset Interface** ([ERC-7943](https://eips.ethereum.org/EIPS/eip-7943)), an emerging standard that enables DeFi protocols to interact with tokenized real-world assets in a standardized way.

### Why ERC-7943 Matters

```
┌─────────────────────────────────────────────────────────────────┐
│                    ERC-7943 BENEFITS                             │
└─────────────────────────────────────────────────────────────────┘

  WITHOUT ERC-7943                   WITH ERC-7943
  ────────────────                   ─────────────
  
  DeFi Protocol                      DeFi Protocol
       │                                  │
       ▼                                  ▼
  "How do I check if               "canTransfer(from, to, amount)"
   this RWA token allows            ↓
   transfers? Every token           Standard response: true/false
   has different methods..."        
                                    Works with ANY RWA token!
```

### Standard Functions Implemented

| ERC-7943 Function | STV3 Implementation |
|-------------------|---------------------|
| `canTransact(account)` | Checks DID existence + KYC verification status |
| `canTransfer(from, to, amount)` | Runs Validation Engine with all configured rules |
| `getFrozenTokens(account)` | Returns frozen balance from LockupFacet |
| `setFrozenTokens(account, amount)` | Freeze investor tokens (`lockTokens`) |
| `forcedTransfer(from, to, amount)` | Legal/regulatory transfer by Recovery Operator |

### Integration with DeFi

ERC-7943 compliance means STV3 tokens can integrate with:

| Platform Type | Integration |
|---------------|-------------|
| **AMM Pools** | Check `canTransfer` before swaps |
| **Lending Protocols** | Verify `canTransact` for borrowers |
| **Custody Platforms** | Standard compliance interface |
| **Tokenization Marketplaces** | Universal RWA detection via ERC-165 |

---

## Compliance Features Summary

| Feature | Purpose |
|---------|---------|
| **DID Verification** | Ensure all holders have verified identity |
| **Trust List** | Whitelist addresses that bypass certain checks |
| **Transfer Rules** | Custom rules for each transfer |
| **Lockup Periods** | Time-based transfer restrictions |
| **Forced Operations** | Regulatory compliance tools |
| **Pause Capability** | Stop trading when needed |
| **Role Separation** | Different permissions for different operations |
| **ERC-7943 Interface** | Universal RWA interoperability |

---

## Related Documentation

- [StoboxRWAVaultFactory](./vault-factory.md) — How tokens are created
- [StoboxDID](./stobox-did.md) — Identity verification system
- [Roles and Permissions](../03-roles-and-permissions.md) — Detailed role descriptions
- [Compliance and Security](../04-compliance-and-security.md) — Security features

---

[← Back to Components](./README.md) | [Previous: Vault Factory](./vault-factory.md) | [Next: Offering Registry →](./offering-registry.md)
