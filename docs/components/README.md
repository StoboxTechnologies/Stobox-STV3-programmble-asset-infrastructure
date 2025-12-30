# System Components

> Detailed description of each platform component

---

## Overview

The Stobox RWA infrastructure is built on four interconnected smart contract systems, each serving a specific purpose in the tokenization ecosystem.

---

## Components

| # | Component | Purpose |
|---|-----------|---------|
| 1 | [StoboxRWAVaultFactory](./vault-factory.md) | Creates and deploys new security tokens |
| 2 | [StoboxProtocolSTV3](./protocol-stv3.md) | The security token itself with all functionality |
| 3 | [StoboxRWAOfferingRegistry](./offering-registry.md) | Manages token offerings and investor purchases |
| 4 | [StoboxDID](./stobox-did.md) | Decentralized identity for investor verification |

---

## How Components Work Together

<!-- TODO: Add interaction diagram -->

---

## Component Dependencies

```
┌─────────────────────────────────────────────────────────────┐
│                    StoboxRWAVaultFactory                    │
│                   (Creates new tokens)                      │
└─────────────────────────┬───────────────────────────────────┘
                          │ creates
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    StoboxProtocolSTV3                       │
│              (Security Token + Treasury)                    │
└───────────┬─────────────────────────────────┬───────────────┘
            │ registers offerings             │ validates identity
            ▼                                 ▼
┌───────────────────────────┐   ┌─────────────────────────────┐
│ StoboxRWAOfferingRegistry │   │        StoboxDID            │
│   (STO Management)        │◄──│  (Investor Verification)    │
└───────────────────────────┘   └─────────────────────────────┘
```

---

[← Back to Index](../README.md) | [Previous: Token Lifecycle](../02-token-lifecycle.md) | [Next: Roles and Permissions →](../03-roles-and-permissions.md)

