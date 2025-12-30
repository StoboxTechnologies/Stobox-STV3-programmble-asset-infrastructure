# Diagrams

> Visual representations and flowcharts

---

## Architecture Diagram

<!-- TODO: Add architecture diagram -->

---

## Token Creation Flow

<!-- TODO: Add flowchart -->

```mermaid
flowchart TD
    A[Client Request] --> B[StoboxRWAVaultFactory]
    B --> C[Create Security Token]
    B --> D[Create Treasury]
    C --> E[StoboxProtocolSTV3]
    D --> F[STV3Treasury]
    E --> G[Configure Token]
    G --> H[Ready for Issuance]
```

---

## Investment Flow

<!-- TODO: Add flowchart -->

```mermaid
flowchart TD
    A[Investor] --> B{Has Valid DID?}
    B -->|No| C[Complete KYC]
    C --> D[Create DID]
    D --> B
    B -->|Yes| E[Select Offering]
    E --> F{Passes Rules?}
    F -->|No| G[Rejected]
    F -->|Yes| H[Make Payment]
    H --> I[Receive Tokens]
    I --> J{Lockup Period?}
    J -->|Yes| K[Tokens Locked]
    J -->|No| L[Tokens Available]
    K --> M[Wait for Unlock]
    M --> L
```

---

## Offering Lifecycle

<!-- TODO: Add state diagram -->

```mermaid
stateDiagram-v2
    [*] --> SCHEDULED
    SCHEDULED --> ACTIVATED: activate
    ACTIVATED --> PAUSED: pause
    PAUSED --> ACTIVATED: unpause
    ACTIVATED --> SUSPENDED: suspend
    SUSPENDED --> ACTIVATED: unsuspend
    ACTIVATED --> COMPLETED: hard cap reached
    ACTIVATED --> TERMINATED: terminate
    SCHEDULED --> CANCELED: cancel
    COMPLETED --> [*]
    TERMINATED --> [*]
    CANCELED --> [*]
```

---

## Component Interaction

<!-- TODO: Add sequence diagram -->

---

## Role Hierarchy

<!-- TODO: Add hierarchy diagram -->

---

[← Back to Index](./README.md) | [Previous: Compliance and Security](./05-compliance-and-security.md)

