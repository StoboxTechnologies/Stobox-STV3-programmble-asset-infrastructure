# StoboxRWAVaultFactory

> Token creation factory — the starting point for tokenizing any real-world asset

---

## Overview

**StoboxRWAVaultFactory** is the smart contract responsible for creating new security tokens on the Stobox platform. Think of it as a "token factory" — when a company decides to tokenize their asset (real estate, equity, bonds, etc.), this is where the token gets born.

Every security token created through Stobox originates from this factory, ensuring standardization, security, and compliance across all tokenized assets.

---

## Purpose

The Factory serves several critical functions:

| Function | Description |
|----------|-------------|
| **Token Creation** | Deploys new security tokens with customizable parameters |
| **Treasury Deployment** | Automatically creates a dedicated Treasury for each token |
| **Standardization** | Ensures all tokens follow the same proven architecture |
| **Version Control** | Manages different feature packages for token capabilities |
| **Registry** | Maintains a complete list of all created tokens |

---

## How It Works

When a new security token is created, the Factory performs the following steps:

```
┌─────────────────────────────────────────────────────────────┐
│                    TOKEN CREATION FLOW                       │
└─────────────────────────────────────────────────────────────┘

    Client Request                Factory Process
    ──────────────                ───────────────
         │
         ▼
    ┌─────────────┐
    │ Token Name  │
    │ Symbol      │──────────▶  1. Validate parameters
    │ Max Supply  │
    │ Package #   │──────────▶  2. Select feature package
    │ Owner       │
    │ DID         │──────────▶  3. Link to identity system
    └─────────────┘
                                        │
                                        ▼
                               ┌─────────────────┐
                               │ Deploy Security │
                               │     Token       │
                               │ (STV3 Protocol) │
                               └────────┬────────┘
                                        │
                                        ▼
                               ┌─────────────────┐
                               │ Deploy Treasury │
                               │   (Token Vault) │
                               └────────┬────────┘
                                        │
                                        ▼
                               ┌─────────────────┐
                               │ Register Token  │
                               │  in Factory     │
                               └────────┬────────┘
                                        │
                                        ▼
                                ✅ Token Ready
```

---

## What Gets Created

When you create a token through the Factory, you receive **two smart contracts**:

### 1. Security Token (StoboxProtocolSTV3)

The token itself — an ERC-20 compatible security token with:
- Customizable name, symbol, and decimals
- Maximum supply cap
- Built-in compliance and validation
- Role-based access control
- Modular, upgradeable architecture

### 2. Treasury Contract

A dedicated vault for the token that:
- Holds newly issued tokens before distribution
- Manages token reserves
- Handles incoming payments during offerings
- Provides secure withdrawal mechanisms

---

## Token Parameters

When creating a new token, the following parameters are configured:

| Parameter | Description | Example |
|-----------|-------------|---------|
| **Name** | Full name of the token | "Acme Real Estate Token" |
| **Symbol** | Trading ticker (3-5 characters) | "ACME" |
| **Decimals** | Divisibility (typically 18) | 18 |
| **Max Supply** | Maximum tokens that can ever exist | 1,000,000 |
| **Owner** | Address with administrative control | 0x123...abc |
| **uDID** | Unique Decentralized ID for the token issuer | "ACME-2024-001" |
| **Package** | Feature set version | Package #7 |

---

## Packages (Feature Sets)

The Factory offers different **packages** — pre-configured sets of functionality that determine what features your token will have. This allows tokens to be created with exactly the capabilities needed.

### Available Packages

| Package | Version | Key Features |
|---------|---------|--------------|
| **GenesisPackage** | Base | Roles, Monetary, Validation, Emergency |
| **RWAVaultBaseSTV3Pack** | v1.0 | Core functionality for RWA tokens |
| **1_5_0_STV3Pack** | v1.5 | + Purchase capability, + Lockup management |
| **1_6_0_STV3Pack** | v1.6 | + Enhanced monetary operations |
| **1_7_0_STV3Pack** | v1.7 | Latest features, optimized operations |

Each newer package typically includes all features from previous versions plus new capabilities.

### Package Contents

A typical package includes these modules (facets):

| Module | Purpose |
|--------|---------|
| **DiamondCutFacet** | Enables upgrades |
| **DiamondLoupeFacet** | Contract introspection |
| **RolesFacet** | Access control management |
| **MonetaryFacet** | Token issuance and redemption |
| **ValidationFacet** | Transfer compliance rules |
| **PurchaseFacet** | Investor purchase handling |
| **LockupFacet** | Token lockup periods |

---

## Token Creation Example

Here's a simplified view of what happens when creating a token:

**Input:**
```
Name:        "Manhattan Office Tower Token"
Symbol:      "MOTT"  
Decimals:    18
Max Supply:  10,000,000
Owner:       0xABC...123
Package:     7 (1_7_0_STV3Pack)
uDID:        "MOTT-NYC-2024"
```

**Output:**
```
✅ Security Token deployed at: 0x1234...5678
✅ Treasury deployed at:       0x8765...4321
✅ Token registered in Factory
✅ Ready for configuration and issuance
```

---

## After Token Creation

Once a token is created, the following steps typically occur:

1. **Treasury Setup** — Link the Treasury to the token
2. **Role Assignment** — Assign Financial and Recovery operators  
3. **Validation Rules** — Configure compliance rules (e.g., require DID)
4. **Initial Issuance** — Mint tokens to the Treasury
5. **Offering Setup** — Register with the Offering Registry for sales

These steps are covered in detail in the [Token Lifecycle](../02-token-lifecycle.md) documentation.

---

## Key Benefits

| Benefit | Description |
|---------|-------------|
| **One-Click Deployment** | Complete token infrastructure in a single transaction |
| **Standardized Security** | All tokens inherit proven, audited architecture |
| **Automatic Treasury** | No need to deploy vault contracts separately |
| **Version Control** | Choose the right feature set for your needs |
| **Full Traceability** | All tokens registered and traceable to the Factory |
| **Upgrade Path** | Tokens can be upgraded to newer packages later |

---

## Related Documentation

- [StoboxProtocolSTV3](./protocol-stv3.md) — The token created by this factory
- [Token Lifecycle](../02-token-lifecycle.md) — Complete journey from creation to trading
- [Deployed Contracts](../deployed-contracts.md) — Factory addresses on supported networks

---

[← Back to Components](./README.md) | [Next: Protocol STV3 →](./protocol-stv3.md)
