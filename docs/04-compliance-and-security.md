# Compliance and Security

> Regulatory compliance features and security mechanisms built into the Stobox STV3 Protocol

---

## Overview

The Stobox STV3 Protocol embeds compliance directly into the smart contract architecture. Unlike traditional systems where compliance is an external overlay, STV3 makes **non-compliant transactions technically impossible**.

```
┌─────────────────────────────────────────────────────────────────┐
│                   COMPLIANCE LAYERS                              │
└─────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────┐
                    │   TRANSFER REQUEST  │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
    │Token Paused?│    │  Lockups?   │    │Validation   │
    │             │    │             │    │   Rules     │
    └──────┬──────┘    └──────┬──────┘    └──────┬──────┘
           │                   │                   │
           └───────────────────┼───────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  ALL CHECKS PASS?   │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
       ┌─────────────┐                  ┌─────────────┐
       │  ✅ EXECUTE │                  │  ❌ REVERT  │
       │  TRANSFER   │                  │ TRANSACTION │
       └─────────────┘                  └─────────────┘
```

**Key Principles:**

| Principle | Implementation |
|-----------|----------------|
| **Compliance by Design** | Rules enforced at smart contract level |
| **Identity-Bound Assets** | Every holder must have verified DID |
| **Automatic Validation** | Every transfer checked against rules |
| **Trust List Bypass** | Whitelisted addresses skip validation |
| **Emergency Controls** | Authorized recovery operations |

---

## Transfer Validation

Every token transfer passes through the validation engine automatically. This happens on:

- Direct transfers (`transfer()`)
- Approved transfers (`transferFrom()`)
- Treasury distributions
- Purchases through offerings

### Validation Flow

```
┌─────────────────────────────────────────────────────────────────┐
│               TRANSFER VALIDATION FLOW                           │
└─────────────────────────────────────────────────────────────────┘

  TRANSFER REQUEST
        │
        ▼
  ┌─────────────┐     YES
  │Token Paused?├─────────► REVERT: FundsTransferProhibited
  └──────┬──────┘
         │ NO
         ▼
  ┌─────────────────────┐     NO
  │ Sender has enough   ├─────────► REVERT: TryToTransferLockedTokens
  │ unlocked balance?   │
  └──────────┬──────────┘
             │ YES
             ▼
  ┌─────────────────────┐     YES
  │ Sender is Trusted?  ├─────────► Skip sender rules
  └──────────┬──────────┘
             │ NO
             ▼
  ┌─────────────────────┐     YES
  │ Receiver is Trusted?├─────────► Skip receiver rules
  └──────────┬──────────┘
             │ NO
             ▼
    ┌─────────────────┐
    │ Execute ALL     │
    │ Validation Rules│──────► Each rule can REVERT
    └────────┬────────┘
             │ ALL PASS
             ▼
      ✅ TRANSFER EXECUTED
```

### What Gets Validated

| Check | Description | Failure Error |
|-------|-------------|---------------|
| **Token Paused** | Is the entire token frozen? | `FundsTransferProhibited` |
| **Lockup Check** | Does sender have enough unlocked tokens? | `TryToTransferLockedTokens` |
| **Validation Rules** | All linked rules pass for sender/receiver | Rule-specific errors |

---

## Validation Rules

Validation rules are modular smart contracts that enforce specific compliance requirements. Rules can be linked or unlinked from tokens as needed.

### Rule Types

```
┌─────────────────────────────────────────────────────────────────┐
│                    VALIDATION RULES                              │
└─────────────────────────────────────────────────────────────────┘

  TOKEN-LEVEL RULES (STV3)              OFFERING RULES (Registry)
  ─────────────────────────             ─────────────────────────
  
  ┌──────────────────┐                  ┌──────────────────────────┐
  │   HasDIDRule     │                  │   ExcludeUSAandCANRule   │
  │                  │                  │                          │
  │ Validates:       │                  │ Validates:               │
  │ - DID exists     │                  │ - Investor country       │
  │ - DID not expired│                  │ - Blocks USA/Canada      │
  │ - DID not blocked│                  │                          │
  └──────────────────┘                  └──────────────────────────┘
                                        
                                        ┌──────────────────────────┐
                                        │AccreditedUSMinInvestment │
                                        │                          │
                                        │ Validates:               │
                                        │ - Accreditation status   │
                                        │ - Min investment for US  │
                                        │   ($100,000)             │
                                        └──────────────────────────┘
                                        
                                        ┌──────────────────────────┐
                                        │ExcludeNonAccreditedUS    │
                                        │MinInvestmentRule         │
                                        │                          │
                                        │ Validates:               │
                                        │ - Non-accredited US      │
                                        │   investors blocked      │
                                        │ - Accredited min: $25k   │
                                        └──────────────────────────┘
```

### HasDIDRule

**Purpose:** Ensures every token holder has a valid decentralized identity.

| Aspect | Details |
|--------|---------|
| **Applied To** | STV3 Token transfers |
| **Checks** | DID exists, not expired, not blocked |
| **Error** | `NotValidDIDOf(address)` |
| **Bypass** | Trusted addresses |

**How It Works:**
```
For each address (sender and receiver):
  1. Query DID contract: getUserDID(address)
  2. Check: validTo > current timestamp
  3. Check: blocked == false
  4. If any check fails → REVERT
```

### ExcludeUSAandCANRule

**Purpose:** Blocks investors from USA and Canada from participating in offerings.

| Aspect | Details |
|--------|---------|
| **Applied To** | Offering purchases |
| **Checks** | Investor's COUNTRY attribute |
| **Error** | `InvestorFromRestrictedCountry(country, address)` |
| **Bypass** | None (regulatory requirement) |

**How It Works:**
```
1. Get investor's COUNTRY attribute from DID
2. Compare hash: keccak256(investorAddress + "USA")
3. Compare hash: keccak256(investorAddress + "CAN")
4. If match → REVERT: InvestorFromRestrictedCountry
```

### AccreditedUSMinInvestmentRule

**Purpose:** Enforces accreditation requirement with higher minimum investment for US investors.

| Aspect | Details |
|--------|---------|
| **Applied To** | Offering purchases |
| **Checks** | ACCREDITED attribute |
| **Error** | `InvestorIsNotAccredited(address)` |
| **US Minimum** | $100,000 |

**How It Works:**
```
1. Get ACCREDITED attribute from DID
2. If not accredited → REVERT
3. If US investor → set minInvestment = $100,000
4. Otherwise → use offering default minimum
```

### ExcludeNonAccreditedUSMinInvestmentRule

**Purpose:** Allows non-US investors regardless of accreditation, but blocks non-accredited US investors.

| Aspect | Details |
|--------|---------|
| **Applied To** | Offering purchases |
| **Accredited Minimum** | $25,000 |
| **Error** | `NonAccreditedInvestorFromRestrictedCountry(country, address)` |

**How It Works:**
```
1. Check ACCREDITED attribute
2. If accredited:
   - Set minInvestment = $25,000
3. If not accredited:
   - Check COUNTRY attribute
   - If USA → REVERT
   - Otherwise → allow with default minimum
```

---

## Trust List

The Trust List allows specific addresses to bypass validation rules. This is essential for operational addresses that need unrestricted token movement.

### Typical Trust List Members

| Address Type | Purpose |
|--------------|---------|
| **Treasury** | Token reserve operations |
| **Offering Registry** | Distribution to investors |
| **Exchange Contracts** | Future secondary trading |
| **Custodian Addresses** | Institutional custody |

### Trust List Operations

| Operation | Access | Function |
|-----------|--------|----------|
| Add to Trust List | Deployer | `trust(address, reason)` |
| Remove from Trust List | Deployer | `distrust(address, reason)` |
| View Trust List | Public | `trustList()` |
| Check if Trusted | Public | `isTrusted(address)` |

### How Trust List Works

```
┌─────────────────────────────────────────────────────────────────┐
│                  TRUST LIST BYPASS                               │
└─────────────────────────────────────────────────────────────────┘

  SCENARIO: Treasury → Investor Distribution
  
  ┌──────────┐                              ┌──────────┐
  │ TREASURY │ ────── TRANSFER ──────────► │ INVESTOR │
  │ (Trusted)│                              │(Not Trusted)
  └──────────┘                              └──────────┘
        │                                         │
        │                                         │
        ▼                                         ▼
  Skip sender rules                        Apply receiver rules
  (Treasury is trusted)                    (HasDIDRule, etc.)
```

> **Note:** If BOTH sender AND receiver are trusted, all validation rules are skipped entirely.

---

## Token Pause

Token pause is a safety mechanism that immediately halts all token transfers.

### Pause Levels

```
┌─────────────────────────────────────────────────────────────────┐
│                   TOKEN PAUSE LEVELS                             │
└─────────────────────────────────────────────────────────────────┘

  LEVEL 1: Token Pause (ValidationFacet)
  ──────────────────────────────────────
  Controlled by: Deployer
  Functions: pauseToken() / unpauseToken()
  Effect: All transfers blocked
  
  LEVEL 2: Protocol Pause (EmergencyFacet)
  ────────────────────────────────────────
  Controlled by: Recovery Operator
  Functions: pauseProtocolSTV3() / unpauseProtocolSTV3()
  Effect: Same as Level 1 (emergency access)
```

### When to Use Pause

| Scenario | Action |
|----------|--------|
| Security incident detected | Immediate pause |
| Smart contract vulnerability | Pause while investigating |
| Regulatory requirement | Compliance-driven pause |
| Corporate action | Temporary halt during event |

### Events

| Event | When Emitted |
|-------|--------------|
| `TokenPaused(address pausedBy)` | Token paused |
| `TokenUnpaused(address unpausedBy)` | Token unpaused |

---

## Token Lockup

Lockups prevent investors from transferring tokens until a specified date. This is used for:

- Vesting schedules
- Offering lockup periods
- Regulatory holding requirements

### Lockup Structure

```solidity
struct Lockup {
    uint256 amount;      // Tokens locked
    uint256 unlockDate;  // When tokens become transferable
    string note;         // Reason for lockup
}
```

### Lockup Operations

| Operation | Access | Function |
|-----------|--------|----------|
| Lock tokens | Deployer | `lockTokens(investor, amount, unlockDate, note)` |
| Clear all lockups | Deployer | `clearLockupsOf(investor, reason)` |
| View lockups | Public | `listOfLockups(investor)` |
| Total locked amount | Public | `totalLockedAmount(investor)` |
| Available balance | Public | `notLockedBalanceOf(investor)` |

### How Lockups Work

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOCKUP MECHANISM                              │
└─────────────────────────────────────────────────────────────────┘

  INVESTOR BALANCE: 1,000 tokens
  
  LOCKUPS:
  ├── Lockup 1: 500 tokens until 2025-06-01
  └── Lockup 2: 300 tokens until 2025-12-01
  
  CALCULATION (today = 2025-01-01):
  ────────────────────────────────
  Total Balance:      1,000
  Locked Amount:        800 (500 + 300)
  ────────────────────────────────
  Available to Transfer: 200
  
  If investor tries to transfer 300 tokens:
  → REVERT: TryToTransferLockedTokens(investor, 200, 300)
```

---

## Emergency Operations

> ⚠️ **Important Notice**
> 
> The `EmergencyFacet` is **NOT included by default** in standard token deployments. This functionality is provided as an example of the protocol's capabilities for regulatory compliance scenarios.
> 
> Enabling emergency operations for an issuer requires:
> - **Explicit agreement with Stobox**
> - **Documented legal justification** (court orders, regulatory requirements)
> - **Formal approval process**
> 
> These functions exist to meet potential regulatory requirements in certain jurisdictions but are only activated when absolutely necessary.

Emergency operations are privileged functions for exceptional circumstances. These bypass normal validation but are strictly controlled.

### Emergency Functions

| Function | Access | Purpose |
|----------|--------|---------|
| `forcedTransfer()` | Recovery Operator | Move tokens between addresses |
| `forcedMint()` | Recovery Operator | Create new tokens |
| `forcedBurn()` | Recovery Operator | Destroy tokens |
| `pauseProtocolSTV3()` | Recovery Operator | Emergency token pause |
| `unpauseProtocolSTV3()` | Recovery Operator | Resume after emergency |

### Forced Transfer

**Use Cases:**
- Court order requiring token movement
- Recovery from compromised wallet
- Corporate action requiring reallocation

**How It Works:**
```
1. Validation temporarily disabled
2. Tokens transferred: FROM → TREASURY
3. Validation re-enabled
4. Tokens distributed: TREASURY → TO
5. Event emitted with reason
```

### Forced Mint

**Use Cases:**
- Correcting issuance errors
- Legal requirement to issue additional tokens
- Compensation for lost tokens

**How It Works:**
```
1. Issue new tokens to Treasury
2. Transfer from Treasury to recipient
3. Event emitted with reason
```

### Forced Burn

**Use Cases:**
- Token recall
- Investor exit processing
- Regulatory requirement

**How It Works:**
```
1. Validation temporarily disabled
2. Tokens transferred: FROM → TREASURY
3. Validation re-enabled
4. Redeem (burn) tokens from Treasury
5. Event emitted with reason
```

### Emergency Event Logging

All emergency operations emit a `Forced` event:

```solidity
event Forced(
    string description,  // "Transfer", "Mint", or "Burn"
    address from,        // Source address (0x0 for mint)
    address to,          // Destination (0x0 for burn)
    uint256 amount,      // Amount affected
    string reason,       // Explanation (audit trail)
    address forcedBy     // Who executed
);
```

---

## Investor Protection

The protocol includes multiple layers of investor protection:

### Soft Cap Protection

If an offering fails to reach its soft cap:
1. Offering status changes to `FAILED_SOFTCAP`
2. Refund process can be initiated
3. Investors receive their payment tokens back
4. Security tokens are returned to Treasury

### Payment Token Locking

During offerings with soft cap:
```
┌─────────────────────────────────────────────────────────────────┐
│                 PAYMENT TOKEN FLOW                               │
└─────────────────────────────────────────────────────────────────┘

  Before Soft Cap Met:
  ────────────────────
  Payment tokens → LOCKED in Treasury
  Security tokens → LOCKED for investor
  
  After Soft Cap Met:
  ───────────────────
  Payment tokens → AVAILABLE to issuer
  Security tokens → REMAIN locked until lockup expires
  
  If Soft Cap NOT Met:
  ────────────────────
  Payment tokens → REFUNDED to investor
  Security tokens → RETURNED to Treasury
```

### Identity Verification

- Every investor must have verified DID
- DID must not be expired or blocked
- Country and accreditation checks per offering

---

## Audit and Transparency

### On-Chain Events

All significant actions emit events for audit trail:

| Category | Events |
|----------|--------|
| **Validation** | `Trust`, `Distrust`, `ValidationRuleLinked`, `ValidationRuleUnlinked` |
| **Pause** | `TokenPaused`, `TokenUnpaused` |
| **Lockup** | `TokensLocked`, `TokensUnlocked`, `LockupCleared` |
| **Emergency** | `Forced` (with full details) |
| **DID** | All attribute changes, address links |

### Verification Functions

| Function | Returns |
|----------|---------|
| `trustList()` | All trusted addresses |
| `validationRulesList()` | All linked validation rules |
| `tokenPaused()` | Current pause status |
| `listOfLockups(address)` | Investor's lockups |
| `isTrusted(address)` | Trust status check |

### Compliance Reporting

The on-chain data enables:
- Real-time compliance monitoring
- Automated regulatory reporting
- Full transaction audit trail
- Investor eligibility verification

---

## Related Documentation

- [Architecture Overview](./01-architecture-overview.md) — System design
- [Token Lifecycle](./02-token-lifecycle.md) — Complete flow
- [Roles and Permissions](./03-roles-and-permissions.md) — Access control
- [StoboxDID](./components/stobox-did.md) — Identity system

---

[← Back to Index](./README.md) | [Previous: Roles and Permissions](./03-roles-and-permissions.md) | [Next: Diagrams →](./05-diagrams.md)
