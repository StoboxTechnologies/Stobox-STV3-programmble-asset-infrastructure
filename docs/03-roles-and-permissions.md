# Roles and Permissions

> Access control and role management across the platform

---

## Overview

The Stobox STV3 Protocol implements a comprehensive role-based access control (RBAC) system across all its components. This ensures that sensitive operations are restricted to authorized personnel while maintaining operational flexibility.

Each component has its own set of roles tailored to its specific functions, creating a layered security model where different responsibilities are clearly separated.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ROLE-BASED ACCESS CONTROL                               │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
  │   VaultFactory      │   │    STV3 Token       │   │  OfferingRegistry   │
  ├─────────────────────┤   ├─────────────────────┤   ├─────────────────────┤
  │ • Master            │   │ • Deployer          │   │ • Master            │
  │ • Admin             │   │ • Owner             │   │ • Admin             │
  │ • Tech Service      │   │ • Financial Op.     │   │ • Tech Service      │
  └─────────────────────┘   │ • Recovery Op.      │   └─────────────────────┘
                            └─────────────────────┘
                                      │
                                      │ validates
                                      ▼
                            ┌─────────────────────┐
                            │     StoboxDID       │
                            ├─────────────────────┤
                            │ • Default Admin     │
                            │ • Writer            │
                            │ • Attribute Reader  │
                            └─────────────────────┘
```

---

## Token Roles (StoboxProtocolSTV3)

The security token has the most granular role system to ensure secure asset management.

### Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                    STV3 TOKEN ROLES                              │
└─────────────────────────────────────────────────────────────────┘

                        ┌───────────┐
                        │ Deployer  │ ← Highest authority
                        └─────┬─────┘   (can change Owner)
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
        ┌──────────┐   ┌───────────┐   ┌───────────┐
        │  Owner   │   │ Financial │   │ Recovery  │
        │          │   │ Operator  │   │ Operator  │
        └──────────┘   └───────────┘   └───────────┘

  Note: Owner and Deployer share DEFAULT_ADMIN_ROLE permissions,
        but only Deployer can transfer ownership or deployer rights.
```

### Deployer

The highest authority in the token contract — the address that deployed the token.

| Aspect | Details |
|--------|---------|
| **Role ID** | `DEFAULT_ADMIN_ROLE` (0x00) |
| **Assigned To** | `StoboxRWAVaultFactory` contract (by default) |
| **Can Be Changed** | Yes, via `setDeployer()` (only by Deployer) |

> **Note:** By default, the Deployer is the `StoboxRWAVaultFactory` contract address that created the token. This allows the factory to perform initial configuration. Deployer rights can later be transferred to another address if needed.

**Permissions:**
- **Transfer ownership** to another address (`transferOwnership()`)
- **Transfer deployer rights** to another address (`setDeployer()`)
- Grant and revoke FINANCIAL_OPERATOR and RECOVERY_OPERATOR roles
- Upgrade token functionality (Diamond cuts)
- **Configuration:** `setTreasury()`, `setMaxIssuance()`, `setOfferingRegistry()`
- **Validation:** `trust()`, `distrust()`, `linkRule()`, `unlinkRule()`
- **Lockup:** `lockTokens()`, `clearLockupsOf()`
- **Control:** `pauseToken()`, `unpauseToken()`

### Owner

The business owner of the token — typically the token issuer.

| Aspect | Details |
|--------|---------|
| **Role ID** | `DEFAULT_ADMIN_ROLE` (shared with Deployer) |
| **Assigned To** | Token issuer (company wallet) |
| **Can Be Changed** | Yes, via `transferOwnership()` (only by Deployer) |

**Permissions:**
- Grant and revoke FINANCIAL_OPERATOR and RECOVERY_OPERATOR roles
  - ❌ Cannot change Owner
  - ❌ Cannot change Deployer

### Financial Operator

Manages monetary operations for the token via `MonetaryFacet`.

| Aspect | Details |
|--------|---------|
| **Role ID** | `FINANCIAL_OPERATOR` |
| **Assigned To** | Treasury manager, CFO wallet |
| **Can Be Changed** | Yes, by Owner or Deployer |

**Permissions:**
- `issue()` — mint new tokens to treasury
- `redeem()` — burn tokens from treasury
- `transferFromTreasury()` — transfer tokens from treasury
- `transferFromTreasuryWithLockupPossibility()` — transfer with optional lockup
- `withdrawERC20FromTreasury()` — withdraw payment tokens (e.g., USDC)

### Recovery Operator

Handles emergency and compliance-related operations via `EmergencyFacet`.

| Aspect | Details |
|--------|---------|
| **Role ID** | `RECOVERY_OPERATOR` |
| **Assigned To** | Compliance officer, legal team |
| **Can Be Changed** | Yes, by Owner or Deployer |

**Permissions:**
- `forcedTransfer()` — regulatory compliance transfers
- `forcedMint()` — emergency token minting
- `forcedBurn()` — emergency token burning
- `pauseProtocolSTV3()` / `unpauseProtocolSTV3()` — emergency protocol pause

---

## Factory Roles (StoboxRWAVaultFactory)

The VaultFactory manages token creation and post-deployment configuration.

### Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                  VAULT FACTORY ROLES                             │
└─────────────────────────────────────────────────────────────────┘

                        ┌───────────┐
                        │  Master   │ ← Stobox platform
                        └─────┬─────┘
                              │ grantInitialAdmin()
                              ▼
                        ┌──────────┐
                        │  Admin   │ ← DEFAULT_ADMIN_ROLE
                        └─────┬────┘
                              │
                              ▼
                       ┌─────────────┐
                       │Tech Service │
                       └─────────────┘
```

### Master

The supreme authority over the factory contract itself.

| Aspect | Details |
|--------|---------|
| **Role ID** | Diamond contract owner (`master`) |
| **Assigned To** | Stobox platform |
| **Can Be Changed** | Yes, via `setMaster()` (only by Master) |

**Permissions:**
- `setMaster()` — transfer master rights
- `setFactoryVersion()` — update factory version
- `grantInitialAdmin()` — grant DEFAULT_ADMIN_ROLE to self

### Admin

Administrative role for all factory and token management operations.

| Aspect | Details |
|--------|---------|
| **Role ID** | `DEFAULT_ADMIN_ROLE` |
| **Assigned To** | Platform administrators (granted by Master) |
| **Can Be Changed** | Yes, via `grantRole()` / `revokeRole()` |

**Permissions (CreateSTV3Facet):**
- `setTokenVersionToCreate()` — set version for new tokens
- `setInitializer()` — set initializer contract

**Permissions (TokenBlueprintFacet):**
- `addTokenFacet()` — add new facet blueprint
- `addPackage()` — create deployment package

**Permissions (STV3TokenManagementFacet):**
- `updateFacetsOfSTV3Token()` — upgrade deployed tokens
- `pauseSTV3Token()` / `unpauseSTV3Token()` — pause/unpause tokens
- `trustForSTV3Token()` / `distrustForSTV3Token()` — manage trust lists
- `linkRuleForSTV3Token()` / `unlinkRuleForSTV3Token()` — manage validation rules
- `setMaxIssuanceForSTV3Token()` — set issuance limits
- `setOfferingRegistryForSTV3Token()` — set offering registry
- `lockTokensForSTV3TokenInvestor()` / `clearLockupsOfSTV3TokenInvestor()` — manage lockups
- `updateTreasuryForSTV3Token()` — update treasury contract

**Permissions (DiamondCutFacet):**
- `diamondCut()` — upgrade factory itself

### Tech Service Role

Technical operations for token creation and setup.

| Aspect | Details |
|--------|---------|
| **Role ID** | `TECH_SERVICE_ROLE` |
| **Assigned To** | Backend services, automation |
| **Can Be Changed** | Yes, by Admin |

**Permissions:**
- `createSecurityToken()` — deploy new STV3 token + treasury
- `initialSetup()` — configure token after creation (treasury, roles, rules)

---

## Registry Roles (StoboxRWAOfferingRegistry)

The Offering Registry manages STOs (Security Token Offerings) with lifecycle and refund operations.

### Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│               OFFERING REGISTRY ROLES                            │
└─────────────────────────────────────────────────────────────────┘

                        ┌───────────┐
                        │  Master   │ ← Stobox platform
                        └─────┬─────┘
                              │ grantInitialAdmin()
                              ▼
                        ┌──────────┐
                        │  Admin   │ ← DEFAULT_ADMIN_ROLE
                        └─────┬────┘
                              │
                              ▼
                       ┌─────────────┐
                       │Tech Service │
                       └─────────────┘
```

### Master

The supreme authority over the registry contract itself.

| Aspect | Details |
|--------|---------|
| **Role ID** | Diamond contract owner (`master`) |
| **Assigned To** | Stobox platform |
| **Can Be Changed** | Yes, via `setMaster()` (only by Master) |

**Permissions:**
- `setMaster()` — transfer master rights
- `setRegistryVersion()` — update registry version
- `grantInitialAdmin()` — grant DEFAULT_ADMIN_ROLE to self

### Admin

Administrative role for registry configuration.

| Aspect | Details |
|--------|---------|
| **Role ID** | `DEFAULT_ADMIN_ROLE` |
| **Assigned To** | Platform administrators (granted by Master) |
| **Can Be Changed** | Yes, via `grantRole()` / `revokeRole()` |

**Permissions (RegistryStorageFacet):**
- `setDefaultPurchaseTokenAddresses()` — set accepted payment tokens
- `forcedSetOfferingStatus()` — force change offering status

**Permissions (RegistryRuleEngineFacet):**
- `connectRegistryRule()` — connect global registry rule

**Permissions (DiamondCutFacet):**
- `diamondCut()` — upgrade registry itself

### Tech Service Role

Technical operations for offering management.

| Aspect | Details |
|--------|---------|
| **Role ID** | `TECH_SERVICE_ROLE` |
| **Assigned To** | Backend services, automation |
| **Can Be Changed** | Yes, by Admin |

**Permissions (OfferingGovernanceFacet):**
- `preMintOfferingCreate()` — create new offering
- `setOfferingLockup()` — configure lockup period
- `setOfferingSoftCap()` — set soft cap
- `activateOffering()` / `pauseOffering()` / `unpauseOffering()` — manage offering state
- `cancelOffering()` / `suspendOffering()` / `unsuspendOffering()` — control offering
- `terminateOffering()` / `archiveOffering()` — end offering

**Permissions (RegistryRefundFacet):**
- `setRefundPurchaseLimit()` — set refund batch limit
- `refundOffering()` — process refunds for failed soft cap
- `refundPurchaseManually()` — manually refund specific purchase

**Permissions (RegistryRuleEngineFacet):**
- `addRuleToOffering()` / `removeRuleFromOffering()` — manage offering rules
- `clearAllRulesFromOffering()` — remove all rules from offering

---

## DID Roles (StoboxDID)

The identity system uses OpenZeppelin's AccessControlEnumerable for role management.

### Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                    STOBOX DID ROLES                              │
└─────────────────────────────────────────────────────────────────┘

                      ┌───────────────┐
                      │ Default Admin │ ← Full control
                      └───────┬───────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
              ┌──────────┐       ┌─────────────────┐
              │  Writer  │       │Attribute Reader │
              └──────────┘       └─────────────────┘
                    │
                    │ Some functions also available to:
                    ▼
              ┌──────────┐
              │DID Owner │ (investor's own DID)
              └──────────┘
```

### Default Admin

| Aspect | Details |
|--------|---------|
| **Role ID** | `DEFAULT_ADMIN_ROLE` (0x00) |
| **Assigned To** | Stobox platform (deployer) |
| **Can Be Changed** | Yes, via `grantRole()` / `revokeRole()` (requires at least 1 admin) |

**Permissions:**
- Grant and revoke all roles (`grantRole()`, `revokeRole()`)
- `setMaxDIDLinkedAddresses()` — set max wallets per DID

### Writer Role

| Aspect | Details |
|--------|---------|
| **Role ID** | `WRITER_ROLE` |
| **Assigned To** | KYC service, compliance team |
| **Can Be Changed** | Yes, by Admin |

**Permissions (onlyRole(WRITER_ROLE)):**
- `createDID()` — create new DID for investor
- `addOrUpdateAttributes()` — add/update KYC attributes
- `addOrUpdateExternalReader()` — manage external readers (with uDID)
- `deleteExternalReader()` — remove external reader (with uDID)
- `prolongateDID()` — extend DID validity
- `blockDID()` / `unBlockDID()` — block/unblock DID
- `removeLinkedAddress()` — remove wallet from DID
- `deactivateDIDAttribute()` — deactivate specific attribute

**Permissions (writerOrDIDOwner — Writer OR investor themselves):**
- `linkAddressToDID()` — link new wallet to DID
- `deactivateAddressOfDID()` — deactivate wallet
- `activateAddressOfDID()` — reactivate wallet

### Attribute Reader Role

| Aspect | Details |
|--------|---------|
| **Role ID** | `ATTRIBUTE_READER_ROLE` |
| **Assigned To** | Verification contracts, validation rules |
| **Can Be Changed** | Yes, by Admin |

**Permissions:**
- `readAttributeList()` — get list of attributes
- `readLinkedAddresses()` — get linked wallets
- `readFullDID()` — get complete DID data

> **Note:** Attribute Reader role provides global read access. Individual DID owners can also grant temporary read access to specific addresses via `addOrUpdateExternalReader()`.

### DID Owner (Investor)

Investors with active DID can perform certain actions on their own DID:

| Action | Requirement |
|--------|-------------|
| `linkAddressToDID()` | Must be linked to same DID, not deactivated |
| `deactivateAddressOfDID()` | Must be linked to same DID, not deactivated |
| `activateAddressOfDID()` | Must be linked to same DID, not deactivated |
| `addOrUpdateExternalReader()` | Own DID (no uDID parameter) |
| `deleteExternalReader()` | Own DID (no uDID parameter) |

---

## Permission Matrix

### STV3 Token Permissions

| Action | Deployer | Owner | Financial Op. | Recovery Op. |
|--------|:--------:|:-----:|:-------------:|:------------:|
| Transfer ownership | ✅ | ❌ | ❌ | ❌ |
| Set deployer | ✅ | ❌ | ❌ | ❌ |
| Grant/revoke roles | ✅ | ✅ | ❌ | ❌ |
| Diamond cuts (upgrade) | ✅ | ❌ | ❌ | ❌ |
| Set treasury | ✅ | ❌ | ❌ | ❌ |
| Set max issuance | ✅ | ❌ | ❌ | ❌ |
| Issue tokens | ❌ | ❌ | ✅ | ❌ |
| Redeem tokens | ❌ | ❌ | ✅ | ❌ |
| Transfer from treasury | ❌ | ❌ | ✅ | ❌ |
| Withdraw ERC20 from treasury | ❌ | ❌ | ✅ | ❌ |
| Lock tokens | ✅ | ❌ | ❌ | ❌ |
| Clear lockups | ✅ | ❌ | ❌ | ❌ |
| Pause/unpause (ValidationFacet) | ✅ | ❌ | ❌ | ❌ |
| Pause/unpause (EmergencyFacet) | ❌ | ❌ | ❌ | ✅ |
| Manage trust list | ✅ | ❌ | ❌ | ❌ |
| Link/unlink validation rules | ✅ | ❌ | ❌ | ❌ |
| Forced transfer | ❌ | ❌ | ❌ | ✅ |
| Forced mint | ❌ | ❌ | ❌ | ✅ |
| Forced burn | ❌ | ❌ | ❌ | ✅ |

### Factory Permissions

| Action | Master | Admin | Tech Service |
|--------|:------:|:-----:|:------------:|
| Set master | ✅ | ❌ | ❌ |
| Set factory version | ✅ | ❌ | ❌ |
| Grant initial admin | ✅ | ❌ | ❌ |
| Grant/revoke roles | ❌ | ✅ | ❌ |
| Diamond cuts (upgrade factory) | ❌ | ✅ | ❌ |
| Create security tokens | ❌ | ❌ | ✅ |
| Initial setup (token config) | ❌ | ❌ | ✅ |
| Set token version to create | ❌ | ✅ | ❌ |
| Add token facets/packages | ❌ | ✅ | ❌ |
| Update facets of STV3 tokens | ❌ | ✅ | ❌ |
| Pause/unpause STV3 tokens | ❌ | ✅ | ❌ |
| Manage STV3 trust lists | ❌ | ✅ | ❌ |
| Manage STV3 validation rules | ❌ | ✅ | ❌ |
| Set STV3 max issuance | ❌ | ✅ | ❌ |
| Lock/unlock STV3 investor tokens | ❌ | ✅ | ❌ |

### Registry Permissions

| Action | Master | Admin | Tech Service |
|--------|:------:|:-----:|:------------:|
| Set master | ✅ | ❌ | ❌ |
| Set registry version | ✅ | ❌ | ❌ |
| Grant initial admin | ✅ | ❌ | ❌ |
| Grant/revoke roles | ❌ | ✅ | ❌ |
| Diamond cuts (upgrade registry) | ❌ | ✅ | ❌ |
| Set default purchase tokens | ❌ | ✅ | ❌ |
| Force set offering status | ❌ | ✅ | ❌ |
| Connect global registry rule | ❌ | ✅ | ❌ |
| Create offerings | ❌ | ❌ | ✅ |
| Set offering lockup/soft cap | ❌ | ❌ | ✅ |
| Activate/pause/unpause offering | ❌ | ❌ | ✅ |
| Cancel/suspend/terminate offering | ❌ | ❌ | ✅ |
| Process refunds | ❌ | ❌ | ✅ |
| Add/remove rules from offering | ❌ | ❌ | ✅ |

### DID Permissions

| Action | Default Admin | Writer | Attribute Reader | DID Owner |
|--------|:-------------:|:------:|:----------------:|:---------:|
| Grant/revoke roles | ✅ | ❌ | ❌ | ❌ |
| Set max linked addresses | ✅ | ❌ | ❌ | ❌ |
| Create DID | ❌ | ✅ | ❌ | ❌ |
| Add/update attributes | ❌ | ✅ | ❌ | ❌ |
| Block/unblock DID | ❌ | ✅ | ❌ | ❌ |
| Prolongate DID | ❌ | ✅ | ❌ | ❌ |
| Remove linked address | ❌ | ✅ | ❌ | ❌ |
| Deactivate attribute | ❌ | ✅ | ❌ | ❌ |
| Link address to DID | ❌ | ✅ | ❌ | ✅ |
| Activate/deactivate address | ❌ | ✅ | ❌ | ✅ |
| Manage external readers (with uDID) | ❌ | ✅ | ❌ | ❌ |
| Manage external readers (own DID) | ❌ | ❌ | ❌ | ✅ |
| Read attributes/full DID | ❌ | ❌ | ✅ | ❌ |

### DID Owner Self-Service Capabilities

Investors with verified DID can perform certain actions on their own identity without requiring Writer role assistance:

```
┌─────────────────────────────────────────────────────────────────┐
│              DID OWNER SELF-SERVICE                              │
└─────────────────────────────────────────────────────────────────┘

  🔗 WALLET MANAGEMENT
  ├── linkAddressToDID()      → Add new wallet to your DID
  ├── deactivateAddressOfDID() → Temporarily disable a wallet
  └── activateAddressOfDID()   → Re-enable a disabled wallet

  👁️ ACCESS CONTROL (who can read your data)
  ├── addOrUpdateExternalReader() → Grant read access to address
  └── deleteExternalReader()       → Revoke read access
```

**Use Cases:**

| Action | When to Use |
|--------|-------------|
| **Link new wallet** | Investor wants to use additional wallet for investments |
| **Deactivate wallet** | Wallet compromised or no longer in use |
| **Activate wallet** | Restore previously deactivated wallet |
| **Add external reader** | Allow third-party service (e.g., portfolio tracker) to verify identity |
| **Delete external reader** | Revoke access from third-party service |

**Limitations:**
- Cannot create new DID (requires Writer)
- Cannot modify KYC attributes (requires Writer)
- Cannot block/unblock entire DID (requires Writer)
- Cannot remove wallet permanently (requires Writer)
- Maximum linked wallets: controlled by Admin (`MAX_DID_LINKED_ADDRESSES`)

---

## Security Best Practices

### Role Assignment Guidelines

| Guideline | Rationale |
|-----------|-----------|
| **Minimum necessary permissions** | Grant only the roles needed for specific tasks |
| **Separate concerns** | Different people/wallets for different roles |
| **Use multisig for Owner** | Critical operations require multiple signatures |
| **Regular audits** | Periodically review who has what roles |
| **Secure key storage** | Hardware wallets for role-holding addresses |

### Role Separation Example

```
┌─────────────────────────────────────────────────────────────────┐
│              RECOMMENDED ROLE SEPARATION                         │
└─────────────────────────────────────────────────────────────────┘

  Company Structure              Role Assignment
  ─────────────────              ───────────────
  
  CEO / Board                    → Owner (multisig)
  CFO / Treasury                 → Financial Operator
  Legal / Compliance             → Recovery Operator
  IT / DevOps                    → Tech Service Role
  
  ⚠️  Avoid: Single person with multiple critical roles
  ✅  Prefer: Clear separation with audit trail
```

---

## Related Documentation

- [Architecture Overview](./01-architecture-overview.md) — System design
- [Token Lifecycle](./02-token-lifecycle.md) — When roles are used
- [Compliance and Security](./04-compliance-and-security.md) — Security features
- [STV3 Protocol](./components/protocol-stv3.md) — Token role details

---

[← Back to Index](./README.md) | [Previous: System Components](./components/README.md) | [Next: Compliance and Security →](./04-compliance-and-security.md)
