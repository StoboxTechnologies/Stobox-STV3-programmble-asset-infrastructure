# Stobox-WEB3-Infrastructure

<img width="2048" height="1544" alt="image" src="https://github.com/user-attachments/assets/ed8dae7f-bcb7-4b12-af0e-20eea58129ad" />


# Infrastructure description

Stobox has a modular smart contract infrastructure for the Stobox RWA (Real World Assets) protocol, with clear separation of responsibilities and interactions between contracts. The system facilitates the creation, management, validation, and registration of tokenized assets and related modules:

---

## 1. [StoboxRWA Vault Factory SC](https://arbiscan.io/address/0x096d75d0501c3b1479ffe15569192cec998223b4#code) (entry point for asset vault creation)

The **StoboxRWAVaultFactory** is a foundational smart contract designed to facilitate the creation and management of tokenized Real-World Asset (RWA) Vaults using the Diamond Standard (EIP-2535). It serves as a secure, upgradeable, and modular entry point for deploying RWA vaults, each potentially representing a distinct real-world asset such as real estate, commodities, or financial instruments.

- **Role:** Main factory contract responsible for deploying and registering new vault instances.
  - Acts as the entry point for creating new components of the protocol
  - Responsible for deploying and initializing StoboxProtocolSTV3 Diamond contracts
  - Provides comprehensive token lifecycle management capabilities
- **Architecture:** Built using **Diamond Standard (EIP-2535)** for modularity and upgradability
  - Enables modular design with reusable facet packages (blueprints)
  - Supports dynamic upgrades without redeploying the entire system
- **Creates:**
  - **STV3Treasury SC** – manages protocol treasury operations for the vault
  - **StoboxProtocolSTV3 SC** – core protocol logic for a specific vault instance (Diamond Standard implementation with facets)
- **Manages:** 
  - Maintains administrative control over deployed components via master role
  - Configures validation rules and trusted addresses for protocol instances
  - Provides post-deployment token management (facet updates, pause controls, trust management)
- **Integration:** Registers & manages vaults inside the StoboxRWAOfferingRegistry SC for comprehensive offering management

### Core Features:
- **Diamond Standard (EIP-2535)**: Modular design and contract upgradability
- **Master Role Access Control**: Dedicated master address with administrative privileges for versioning, role reassignment, and upgrade operations
- **Versioning Support**: Built-in version management (major, minor, patch) for clear tracking of contract evolution
- **Blueprint System**: Reusable token facet packages for consistent vault deployments
- **Token Creation**: Automated STV3 token and treasury deployment with initialization
- **Infrastructure Setup**: Role assignment, treasury configuration, and validation rule linking
- **Post-Deployment Management**: Comprehensive token lifecycle management capabilities

### Factory Structure:

#### **Core Contract (Immutable)**
Contains immutable functions defined directly in the diamond that cannot be replaced or removed:
- `constructor()`, `receive()`, `fallback()`
- `setMaster(address newMaster)` - Updates the master (admin) address
- `setFactoryVersion(uint256 major, uint256 minor, uint256 patch)` - Updates factory version
- `master()` - Returns the address of the master
- `getVersion()` - Returns current semantic version (major, minor, patch)

**Access Control:** `onlyMaster` modifier ensures only the master can execute privileged functions

#### **DiamondCutFacet** (v1.0.1)
Allows the contract owner to add, replace, or remove facets dynamically. Uses LibFactoryRoles for access control.

**Key Function:**
- `diamondCut(FacetCut[] calldata _diamondCut, address _init, bytes calldata _calldata)` - Modifies multiple functions across different facets in one transaction

**FacetCut Actions:**
- **Add**: Maps function selectors to a new facet address (reverts if selector already exists)
- **Replace**: Updates existing function selector mappings (reverts if selector is unset or already points to same address)
- **Remove**: Removes function selectors from mapping (reverts if already unset)

**Access Control:** Requires `DEFAULT_ADMIN_ROLE` via LibFactoryRoles

**Version Changes (v1.0.0 → v1.0.1):**
- Changed access control from master-only to role-based (DEFAULT_ADMIN_ROLE)
- Enhanced security with granular permission management

#### **DiamondLoupeFacet** (v1.0.0)
Provides introspection into the factory diamond's modular structure.

**View Functions:**
- `facets()` - Returns all facets with their function selectors
- `facetFunctionSelectors(address _facet)` - Returns all function selectors for a specific facet
- `facetAddresses()` - Returns all facet addresses used in the diamond
- `facetAddress(bytes4 _functionSelector)` - Returns the facet address for a given function selector (or address(0) if not found)
- `supportsInterface(bytes4 interfaceId)` - ERC-165 interface detection

#### **TokenBlueprintFacet** (v1.0.1)
Provides functionality to register and organize token-related facets into reusable packages (blueprints). Enables adding individual token facets with metadata, grouping them into named packages, and retrieving full package details.

**Blueprint Management:**
- `addTokenFacet(address facetAddress, string calldata facetName, string calldata facetVersion)` - Registers a new token facet with metadata
- `addPackage(string calldata packageName, address[] calldata facetsToAdd)` - Creates a blueprint package with specified facets
- `packageDetails(string calldata packageName)` - Returns facet addresses in a package
- `fullPackageInfo(string calldata packageName)` - Returns comprehensive package info (names, addresses, versions)

**Facet Queries:**
- `tokenFacetDetails(address facetAddress)` - Returns name and version of a facet
- `tokenFacetExists(address facetAddress)` - Checks if facet is registered
- `packageCount()` - Returns total number of blueprint packages

**Access Control:** Only `master` role can register facets and create packages

**Version Changes (v1.0.0 → v1.0.1):**
- Added `fullPackageInfo()` for comprehensive package metadata retrieval
- Enhanced blueprint documentation and organization

#### **CreateSTV3Facet** (v1.1.0)

Handles the creation of Security Tokens (STV3) along with their associated treasury contracts. Allows setting creation parameters like token version and initializer contract. Tracks created tokens and exposes metadata for each deployment.

**Token Creation:**
- `createSecurityToken(string calldata name, string calldata symbol, uint8 decimals, uint256 maxSupply, bytes calldata initCalldata)` - Deploys new STV3 token with treasury
- `setTokenVersionToCreate(uint256 major, uint256 minor, uint256 patch)` - Sets version for newly created tokens
- `setInitializer(address newInitializer)` - Sets initializer contract for token setup

**Token Tracking:**
- `tokenCreationDetails(address tokenAddress)` - Returns deployment details for a specific token
- `tokenVersionToCreate()` - Returns version that will be assigned to next created token
- `tokenInitializer()` - Returns current initializer contract address
- `tokenCount()` - Returns total number of created tokens
- `generalTokenList()` - Returns array of all created token addresses

**Access Control:** Restricted to protocol `master`

**Version Changes (v1.0.0 → v1.0.1):**
- Enhanced token creation tracking and metadata management
- Improved version control for created tokens

#### **SetInfrastructureFacet** (v1.2.0)

Performs initial infrastructure setup for newly created STV3 tokens. Assigns key operational roles, sets the treasury address, and links validation rules. Ensures tokens are fully initialized and compliant with protocol standards.

**Infrastructure Setup:**
- `initialSetup(address tokenAddress, address treasury, address financialOperator, address recoveryOperator, address[] calldata validationRules)` - Performs complete initial setup for a newly created STV3 token

**Setup Operations:**
1. Sets treasury address
2. Grants FINANCIAL_OPERATOR role
3. Grants RECOVERY_OPERATOR role
4. Links validation rules to token
5. Trusts the factory as authorized address

**Access Control:** Callable only by protocol `master`


**Version Changes (v1.0.0 → v1.1.0):**
- Enhanced setup automation and validation
- Improved role assignment logic

#### **STV3TokenManagementFacet** (v1.3.0)

Provides comprehensive post-deployment management functionality for STV3 security tokens. Enables updating token facets, controlling pause states, managing trust/distrust lists, linking validation rules, and setting issuance limits for deployed tokens.

**Facet Management:**
- `updateFacetsOfSTV3Token(address tokenAddress, FacetCut[] calldata _diamondCut, address _init, bytes calldata _calldata)` - Updates facets of an existing STV3 token

**Pause Controls:**
- `pauseSTV3Token(address tokenAddress)` - Pauses token transfers
- `unpauseSTV3Token(address tokenAddress)` - Resumes token transfers

**Trust Management:**
- `trustForSTV3Token(address tokenAddress, address account)` - Adds address to trusted list
- `distrustForSTV3Token(address tokenAddress, address account)` - Removes address from trusted list

**Validation Rules:**
- `linkRuleForSTV3Token(address tokenAddress, address ruleAddress)` - Links validation rule to token
- `unlinkRuleForSTV3Token(address tokenAddress, address ruleAddress)` - Removes validation rule from token

**Monetary Controls:**
- `setMaxIssuanceForSTV3Token(address tokenAddress, uint256 newMaxIssuance)` - Sets maximum issuance limit for token

**Access Control:** Accessible only by `DEFAULT_ADMIN_ROLE` to maintain secure control over token lifecycle management

**Version Changes (v1.0.0 → v1.1.0):**
- Added `setMaxIssuanceForSTV3Token()` for monetary policy management
- Enhanced post-deployment control capabilities

#### **FactoryRolesFacet** (v1.0.0)
Provides role-based access control management for the factory using OpenZeppelin's AccessControl and AccessControlEnumerable standards. Uses LibFactoryRoles library for implementation logic.

**Role Management:**
- `grantRole(bytes32 role, address account)` - Grants a role to an account
- `revokeRole(bytes32 role, address account)` - Revokes a role from an account
- `renounceRole(bytes32 role, address account)` - Allows account to renounce their own role

**Role Queries:**
- `hasRole(bytes32 role, address account)` - Checks if account has a specific role
- `getRoleAdmin(bytes32 role)` - Returns the admin role for a given role
- `getRoleMember(bytes32 role, uint256 index)` - Gets role member by index
- `getRoleMemberCount(bytes32 role)` - Returns number of accounts with a role

**Defined Roles:**
- `DEFAULT_ADMIN_ROLE()` - Administrator role for factory operations

### Factory Libraries:
The factory uses internal Solidity libraries for code reuse between facets:
- **LibDiamond** (v1.0.0) - Diamond pattern implementation, storage management, and access control
- **LibVersion** (v1.0.0) - Semantic versioning support (major, minor, patch)
- **LibBlueprint** (v1.0.1) - Token facet blueprint management and package organization
- **LibCreateSTV3** (v1.1.0) - STV3 token creation logic and deployment tracking
- **LibFactoryRoles** (v1.0.0) - Role-based access control implementation

### Factory Versioning System:

The **StoboxRWAVaultFactory** implements the same Diamond versioning system as the StoboxProtocolSTV3. Each deployment and update is strictly tracked using the **major.minor.patch** format for precise tracking of the factory build state and facet composition changes.

#### **Version Components:**

**major** - Architectural version of the factory
- Fundamental architectural changes to the factory infrastructure
- Expected to remain `1` unless a complete redesign occurs
- Breaking changes to the factory architecture

**minor** - Factory Diamond build version (facet composition)
- Each new logical build of the factory Diamond increments `minor`
- Business logic or governance determines the build composition
- New facets or features added to the factory

**patch** - Local changes to factory facets
- Starts from `0` at initial factory deployment
- Incremented by `+1` for each facet modification via `diamondCut`
- Bug fixes and minor updates to factory facets

#### **Core Contract Versioning:**

The factory's core contract provides master-controlled versioning functions:
- `setFactoryVersion(uint256 major, uint256 minor, uint256 patch)` - Updates factory version (master only)
- `getVersion()` - Returns current semantic version (major, minor, patch)

This versioning system uses **LibVersion** library with Diamond Storage pattern for consistent version tracking across all factory upgrades and modifications.

### Deployment Details:

**Arbitrum One Mainnet:**
- **Chain ID:** 42161
- **Address:** [`0x096D75d0501c3B1479FFe15569192CeC998223b4`](https://arbiscan.io/address/0x096d75d0501c3b1479ffe15569192cec998223b4#code)
- **Name:** StoboxRWAVaultFactory
- **Version:** 1.0.1
- **Network Status:** arbiscan.freshstatus.io
- **Repository:** [Stobox-WEB3-Infrastructure](https://github.com/StoboxTechnologies/Stobox-WEB3-Infrastructure)
- **Release Notes:** [StoboxRWAVaultFactory-Release-Notes.md](./StoboxRWAVaultFactory-Release-Notes.md)

**Deployed Facets (Arbitrum Mainnet):**


| Facet | Address | Version |
|-------|---------|---------|

**[StoboxRWAVaultFactory v.1.0.7](https://arbiscan.io/address/0x096d75d0501c3b1479ffe15569192cec998223b4#code) 0x096D75d0501c3B1479FFe15569192CeC998223b4**

**[DiamondInit v.1.0.0](https://arbiscan.io/address/0x3d353efc33c380b739f351FC0d65BA9590aeE0d1#code) 0x3d353efc33c380b739f351FC0d65BA9590aeE0d1**

**[DiamondCutFacet v.1.0.1](https://arbiscan.io/address/0x94A217b601cAfc1eBE7A4DEDdAd0a3b72b8AA5eD#code) 0x94A217b601cAfc1eBE7A4DEDdAd0a3b72b8AA5eD**

**[DiamondLoupeFacet v.1.0.0](https://arbiscan.io/address/0x9051da5380980fcf7753aee624bc5f74eb2f81bb#code) 0x9051Da5380980FCF7753AEe624Bc5f74Eb2f81BB**

**[TokenBlueprintFacet v.1.0.1](https://arbiscan.io/address/0xBBE85938e5C17FA457f8546FDAF8f35f2024D0DF#code) 0xBBE85938e5C17FA457f8546FDAF8f35f2024D0DF**

**[CreateSTV3Facet v.1.1.0](https://arbiscan.io/address/0x6608547a9bb1f28ffbcad8fed9c9acc6e900d826#code) 0x6608547a9bb1f28ffbcad8fed9c9acc6e900d826**

**[SetInfrastructureFacet v.1.2.0](https://arbiscan.io/address/0x5f4f348575aa3d5be91137357562a2b4c8071030#code) 0x5f4f348575aa3d5be91137357562a2b4c8071030**

**[FactoryRolesFacet v.1.0.0](https://arbiscan.io/address/0xA6c669a0Ff236Ba3B7cccd3eA75D007f931F271f#code) 0xA6c669a0Ff236Ba3B7cccd3eA75D007f931F271f**

**[STV3TokenManagementFacet v.1.3.0](https://arbiscan.io/address/0xf1FA8f08d9571b5e489A87f3C7Df42287293c66c#code) 0xf1FA8f08d9571b5e489A87f3C7Df42287293c66c**


**Arbitrum Sepolia Testnet:**
- **Chain ID:** 421614
- **Address:** `0xa1F864848e821D43bD8dDA123ffD47E92c558698`
- **Name:** Factory
- **Network Status:** arbiscan.freshstatus.io

---

## 2. [STV3Treasury SC](https://arbiscan.io/address/0x17b94f6b0ddeaac3753b33ee006a7fdbc1292dcf#code) (financial management)

- **Role:** Handles treasury functions
  - Stores and manages funds or tokenized assets related to the RWA protocol.
  - Ensures secure handling of protocol-related assets.
- **Created by:** StoboxRWAVaultFactory SC.
- **Connections:** Works together with StoboxProtocolSTV3 SC for vault-related operations.

---

## 3. [StoboxProtocolSTV3 SC](https://arbiscan.io/address/0x998a0beaf37ca4ba61b5cfac59fdee0da2211a46#code) (core protocol logic)

- **Role:** STV3 represents Stobox's latest proprietary tokenization standard, engineered to deliver a more secure, efficient, and scalable framework for managing tokenized assets.
- **Architecture:** Built using **Diamond Standard (EIP-2535)** for modularity and upgradability.
  - Enables modular design by separating functionality into different facets
  - Contract logic is encapsulated within a single contract using DELEGATECALL to invoke facet contracts
  - Supports dynamic upgrades without redeploying the entire system
- **Token Standard:** ERC-20 compliant with extended functionality
- **Created by:** StoboxRWAVaultFactory SC
- **Managed:** Directly by the factory contract
- **Works with:** Validation HasDIDRule SC for identity verification

### Core Features:
- **Diamond Standard (EIP-2535)**: Modular design and contract upgradability
- **ERC-20 Token Standard**: Standard token functionalities
- **Role-Based Access Control**: Grants specific addresses permission to mint and burn tokens
- **Validation Management**: Trusted addresses, validation rules, and pre-transfer validation checks
- **Treasury Operations**: Token withdrawals, allowances management, metadata access
- **Monetary Controls**: Issue, redeem, and transfer tokens from treasury
- **Emergency Controls**: Forceful transfers, minting, burning, and protocol pause capabilities

### Facets Structure:

> **What is a Facet?** A facet is a stateless smart contract or Solidity library with external functions. Facets are deployed and their functions are added to diamonds. Facets don't store data in their own storage but can define state and read/write to diamond storage. The term "facet" comes from the diamond industry - it's a side or flat surface of a diamond.

#### **Core Contract (Immutable)**
Contains immutable functions defined directly in the diamond that cannot be replaced or removed:
- `constructor()`, `receive()`, `fallback()`
- `transfer()`, `approve()`, `transferFrom()`
- `setDeployer()`, `transferOwnership()`
- `name()`, `symbol()`, `decimals()`, `totalSupply()`, `balanceOf()`, `maxSupply()`, `allowance()`
- `deployer()`, `owner()`, `getVersion()`

#### **DiamondCutFacet** (v1.0.0)
Enables modifying the diamond by adding, replacing, or removing function selectors. The diamond contains a mapping of function selectors to facet addresses, and this facet allows updating that mapping in a single transaction to prevent data corruption.

**Key Function:**
- `diamondCut(FacetCut[] calldata _diamondCut, address _init, bytes calldata _calldata)` - Modifies multiple functions across different facets in one transaction

**FacetCut Actions:**
- **Add**: Maps function selectors to a new facet address (reverts if selector already exists)
- **Replace**: Updates existing function selector mappings (reverts if selector is unset or already points to same address)
- **Remove**: Removes function selectors from mapping (reverts if already unset)

**Access Control:** Only deployer can execute via `enforceIsDeployer()`

#### **DiamondLoupeFacet** (v1.0.0)
A "loupe" is a small magnifying glass used to examine diamonds. This facet provides introspection into the diamond's modular structure.

**View Functions:**
- `facets()` - Returns all facets with their function selectors
- `facetFunctionSelectors(address _facet)` - Returns all function selectors for a specific facet
- `facetAddresses()` - Returns all facet addresses used in the diamond
- `facetAddress(bytes4 _functionSelector)` - Returns the facet address for a given function selector (or address(0) if not found)
- `supportsInterface(bytes4 interfaceId)` - ERC-165 interface detection

#### **ValidationFacet** (v1.0.1)
Implements comprehensive on-chain validation management and trust relationships for tokens. Typically configured by the deployer (StoboxRWAVaultFactory).

**Trust Management:**
- `trust(address account)` - Adds an address to the trusted list
- `distrust(address account)` - Removes an address from the trusted list
- `trustList()` - Returns array of all trusted addresses
- `isTrusted(address account)` - Checks if an address is trusted

**Rule Management:**
- `linkRule(address ruleAddress)` - Links a validation rule to the token
- `unlinkRule(address ruleAddress)` - Removes a validation rule from the token
- `ruleDescription(address ruleAddress)` - Returns description of a validation rule
- `validationRulesList()` - Returns array of all active validation rules

**Token Controls:**
- `pauseToken()` - Pauses token transfers
- `unpauseToken()` - Resumes token transfers
- `tokenPaused()` - Returns current pause status

**Validation Hooks:**
- `beforeUpdateValidation()` - Pre-transfer validation logic
- `afterUpdateValidation()` - Post-transfer validation logic

**Access Control:** Only deployer can manage trust, rules, and pause state

**Version Changes (v1.0.0 → v1.0.1):**
- Enhanced protocol security with pause/unpause capabilities
- Improved version documentation and contract traceability

#### **RolesFacet** (v1.0.0)
Provides role-based access control management using OpenZeppelin's AccessControl and AccessControlEnumerable standards. Uses LibRoles library for implementation logic.

**Role Management:**
- `initIERC165Roles()` - Initializes ERC-165 support for role interfaces
- `grantRole(bytes32 role, address account)` - Grants a role to an account
- `revokeRole(bytes32 role, address account)` - Revokes a role from an account
- `renounceRole(bytes32 role, address account)` - Allows account to renounce their own role

**Role Queries:**
- `hasRole(bytes32 role, address account)` - Checks if account has a specific role
- `getRoleAdmin(bytes32 role)` - Returns the admin role for a given role
- `getRoleMember(bytes32 role, uint256 index)` - Gets role member by index
- `getRoleMemberCount(bytes32 role)` - Returns number of accounts with a role
- `getRoleMembers(bytes32 role)` - Returns array of all accounts with a role

**Defined Roles:**
- `FINANCIAL_OPERATOR()` - Role for monetary operations (issue, redeem, transfers)
- `RECOVERY_OPERATOR()` - Role for emergency operations (forced actions, pause/unpause)

#### **MonetaryFacet** (v1.1.0)
Implements treasury management and monetary operations with issuance limit controls. Uses LibMonetary, LibRoles, and LibDiamond libraries.

**Treasury Management:**
- `setTreasury(address newTreasury)` - Sets the treasury address (deployer only)
- `treasury()` - Returns current treasury address

**Monetary Operations:**
- `issue(uint256 amount)` - Issues new tokens to treasury (FINANCIAL_OPERATOR only)
- `redeem(uint256 amount)` - Burns tokens from treasury (FINANCIAL_OPERATOR only)
- `transferFromTreasury(address to, uint256 amount)` - Transfers tokens from treasury (FINANCIAL_OPERATOR only)

**Issuance Controls:**
- `setMaxIssuance(uint256 newMaxIssuance)` - Sets maximum token issuance limit (deployer only)
- `maxIssuance()` - Returns current maximum issuance limit
- `totalIssued()` - Returns cumulative tokens issued

**Access Modifiers:**
- `onlyDeployer` - Enforced via LibDiamond.enforceIsDeployer()
- `onlyFinOps` - Enforced via LibRoles._checkRole(FINANCIAL_OPERATOR)

**Version Changes (v1.0.0 → v1.1.0):**
- Added issuance limit controls (`setMaxIssuance`, `maxIssuance`)
- Added issuance tracking (`totalIssued`)
- Enhanced monetary policy enforcement and regulatory compliance support
- Expanded from 5 to 8 total functions

#### **EmergencyFacet** (v1.0.0)
Provides privileged emergency operations for critical recovery scenarios. All functions are restricted to the RECOVERY_OPERATOR role.

**Emergency Operations:**
- `forcedTransfer(address from, address to, uint256 amount)` - Forcefully transfers tokens between addresses
- `forcedMint(address to, uint256 amount)` - Mints tokens directly to an address
- `forcedBurn(address from, uint256 amount)` - Burns tokens from an address

**Protocol Controls:**
- `pauseProtocolSTV3()` - Pauses the entire protocol
- `unpauseProtocolSTV3()` - Resumes protocol operations

**Access Control:** All functions require RECOVERY_OPERATOR role via LibRoles

### Protocol Libraries:
The protocol uses internal Solidity libraries for code reuse between facets:
- **LibDiamond** - Diamond pattern implementation, ownership, and access control
- **LibERC20** - ERC-20 token logic and storage
- **LibMonetary** - Treasury and monetary operations logic
- **LibRoles** - Role-based access control implementation
- **LibValidator** - Validation rules and compliance checks
- **LibVersion** - Semantic versioning support for Diamond builds

### Diamond Versioning System:

Each deployment and update of the StoboxProtocolSTV3 Diamond contract is strictly tracked using a structured versioning scheme in the format **major.minor.patch**. This allows precise tracking of the build state, its structure, and the change history at the facet level.

#### **Version Components:**

**major** - Architectural version of the project
- Represents fundamental architectural changes
- Expected to remain `1` for an extended period unless a complete redesign or architectural overhaul is introduced
- Indicates breaking changes to the core architecture

**minor** - Version of the Diamond build (facet composition)
- Each new logical build of the Diamond (a set of specific facets forming a working system) increments `minor`
- The business logic or project governance determines the composition of the build
- Indicates new features or facet additions that maintain backward compatibility

**patch** - Local changes to facets
- Starts from `0` at the moment of initial Diamond deployment
- For each facet added via `diamondCut` during the initial build process, `patch` is incremented by `+1`
- Any modification to facets (addition, update, or removal via `diamondCut`) increments `patch` by `+1`
- Indicates bug fixes or minor updates

#### **Example Versioning Sequence:**

1. **Diamond deployment** → version = `1.0.0`
2. **Adding facets via diamondCut** (initial build) → version = `1.0.1`
3. **Updating one of the facets** via diamondCut → version = `1.0.2`
4. **Creating a new build** (e.g., with new business logic) → version = `1.1.0`
5. **Further changes** (via any diamondCut operation) → version = `1.1.1`, `1.1.2`, and so on

#### **LibVersion Implementation:**

The versioning logic is implemented in the **LibVersion** library and uses a fixed storage slot (via `keccak256("stobox.version.storage")`) for compatibility with the Diamond Storage pattern (EIP-2535).

**Data Structure:**
```solidity
struct Version {
    uint256 major;    // Architectural version of the project
    uint256 minor;    // Version of the Diamond build (facet composition)
    uint256 patch;    // Local changes to facets
}
```

**Functions:**
- `setVersion(uint256 major, uint256 minor, uint256 patch)` - Sets the full semantic version (used during initial deployment)
- `increasePatch()` - Increments the patch version by +1 and emits VersionUpdated event (used after each diamondCut operation)
- `getVersion()` - Returns the current semantic version as a structured object

**Event:**
- `VersionUpdated(uint256 indexed major, uint256 indexed minor, uint256 indexed patch)` - Emitted when the version is updated

### Deployment Details:
- **Network:** Arbitrum Mainnet
- **Chain ID:** 42161
- **Name:** Stobox Security Token v.3
- **Symbol:** STBX
- **Decimals:** 6
- **Network Status:** arbiscan.freshstatus.io
- **Release Notes:** [StoboxProtocolSTV3-Release-Notes.md](./StoboxProtocolSTV3-Release-Notes.md)

---

## 4. [Validation HasDIDRule SC](https://arbiscan.io/address/0x85a3eae3cd8cecac03d2fc44001e030621320acb#code) (identity-based rule check)

- **Role:** A validation module
  - Implements a specific compliance rule: Has DID (Decentralized Identifier).
  - Ensures that only entities with a valid DID can perform certain actions within the protocol.
- **Interaction:**
  - Invoked by **StoboxProtocolSTV3 SC** during operations that require DID verification.
  - Performs **CHECK** operations against the **StoboxDID SC**.

---

## 5. [StoboxDID SC](https://arbiscan.io/address/0x25E6036178656b1329ee51696407b367D8C6ba84#code) (decentralized identity registry)

The **StoboxDID** smart contract implements a comprehensive decentralized identity (DID) system that enables users to create, manage, and update digital identities on-chain. As real-world asset (RWA) tokenization accelerates, the demand for robust compliance and privacy-centric identity verification grows. The Stobox DID framework directly addresses these challenges by providing secure, blockchain-based identity management.

- **Role:** Stores and manages DIDs (Decentralized Identifiers) for ecosystem participants
  - Manages DID creation, attributes, linked addresses, and access permissions
  - Provides the source of truth for identity verification across the ecosystem
  - Supports secure identity management across multiple addresses
  - Enables controlled data sharing within a decentralized ecosystem
- **Interaction:** Queried by Validation HasDIDRule SC to confirm DID ownership and validity
- **Repository:** [ST4DIDSC](https://github.com/StoboxTechnologies/ST4DIDSC)

### Core Features:
- **Unique Identifier Management**: Each DID is associated with a unique UDID (Unique Decentralized Identifier)
- **Attribute Storage**: Store and manage flexible attributes with validation periods and type metadata
- **Multi-Address Linking**: Support linking multiple wallet addresses to the same DID
- **Role-Based Access Control**: Granular permissions via WRITER_ROLE, ATTRIBUTE_READER_ROLE, and ownership
- **External Reader System**: Time-limited, controlled access for third-party data readers
- **DID Lifecycle Management**: Create, update, prolong, block/unblock DIDs
- **Address Management**: Link, deactivate, activate, and remove addresses from DIDs
- **Attribute Management**: Add, update, deactivate attributes with expiration control
- **Maximum Address Limits**: Configurable maximum number of linked addresses per DID

### Data Structures:

#### **Attribute Structure**
Manages the history and state of DID attributes with flexible data storage.

```solidity
struct Attribute {
    bytes value;              // Attribute value in arbitrary byte format
    string valueType;         // Human-readable type (e.g., "STRING", "UINT", "HASH")
    uint256 createdAt;       // Creation timestamp
    uint256 updatedAt;       // Last update timestamp
    uint256 validTo;         // Validity expiration timestamp
    address lastUpdatedBy;   // Address that last updated the attribute
}
```

**Purpose:**
- Managing attribute history and state
- Verifying data validity based on `validTo` field
- Tracking the last updater via `lastUpdatedBy`
- Storing values in flexible format using `bytes value`

#### **DID Structure**
Represents a complete Decentralized Identity object with comprehensive metadata.

```solidity
struct DID {
    string UDID;                                    // Unique Decentralized Identifier
    uint256 validTo;                                // DID validity expiration timestamp
    uint256 updatedAt;                              // Last update timestamp
    bool blocked;                                   // Block status (suspended/revoked)
    address lastUpdatedBy;                          // Address that last modified DID
    string[] attributeList;                         // List of attribute keys
    mapping(string => Attribute) attributes;        // Attribute key to data mapping
    mapping(address => uint256) externalReader;     // External reader access expiration
}
```

**Purpose:**
- Unique identifier (UDID) management
- Validity control via `validTo` and `blocked` flags
- Attribute storage and tracking through mapping and key list
- Access control for external data readers with expiration
- Auditability via timestamps and `lastUpdatedBy`

#### **Linker Structure**
Manages relationships between addresses and a DID.

```solidity
struct Linker {
    string UDID;                    // Associated Unique Decentralized Identifier
    uint256 joinDate;               // Timestamp when address was linked
    uint256 updateDate;             // Last modification timestamp
    bool deactivated;               // Deactivation status
    address[] linkedAddresses;      // All addresses linked to this UDID
}
```

**Purpose:**
- Track when linking was initiated (`joinDate`)
- Track last modification (`updateDate`)
- Manage active/deactivated link status
- Maintain list of all Ethereum addresses associated with UDID

#### **ParamAttribute Structure**
Input parameter structure for batch attribute operations.

```solidity
struct ParamAttribute {
    address walletAddress;      // Target wallet address
    string attributeName;       // Attribute key/name
    bytes value;                // Attribute value in bytes
    string valueType;           // Human-readable type description
    uint256 validToData;        // Validity expiration timestamp
}
```

**Purpose:** Used in `addOrUpdateAttributes` function to apply or update attribute values for specific wallet addresses.

### Access Control:

#### **DEFAULT_ADMIN_ROLE**
Holds complete control over the role-based access control system. Automatically assigned to contract deployer.

**Capabilities:**
- Grant any role (including DEFAULT_ADMIN_ROLE) to any address
- Revoke any role from any address
- Change role admin for other roles using `_setRoleAdmin`
- Full self-management permissions

**Assignment:** Granted upon contract creation through constructor:
```solidity
constructor() {
    _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
}
```

#### **WRITER_ROLE**
Essential role responsible for managing and updating DIDs and their associated data.

**DID Management:**
- `createDID()` - Creates a new decentralized identifier
- `prolongateDID()` - Extends the validity of an existing DID
- `blockDID()` - Blocks a DID, restricting access or usage
- `unBlockDID()` - Unblocks a previously blocked DID

**Address Linking:**
- `linkAddressToDID()` - Links an address to a specific DID
- `removeLinkedAddress()` - Removes a linked address from a DID
- `deactivateAddressOfDID()` - Deactivates a linked address
- `activateAddressOfDID()` - Reactivates a previously deactivated address

**External Reader Management:**
- `addOrUpdateExternalReader()` - Adds or updates external reader access
- `deleteExternalReader()` - Removes external reader's access

**Attribute Management:**
- `addOrUpdateAttributes()` - Adds or updates DID attributes
- `deactivateDIDAttribute()` - Deactivates an attribute

#### **ATTRIBUTE_READER_ROLE**
Grants view-only access to DID-related data without modification rights.

**DID Data Access:**
- `readFullDID()` - Reads complete DID data including attributes and linked addresses
- `readAttributeList()` - Reads the list of attributes associated with a DID
- `readLinkedAddresses()` - Reads the list of addresses linked to a DID

**Restrictions:** Read-only access, no modification or deletion permissions.

#### **DID Owner**
Any address that is linked to a specific DID. Each linked address is considered an owner of that DID.

**Owner Capabilities:**
- `linkAddressToDID()` - Link another address to their own DID
- `deactivateAddressOfDID()` - Deactivate an address linked to their own DID
- `activateAddressOfDID()` - Reactivate a deactivated address on their own DID
- `addOrUpdateExternalReader()` - Add or update external readers for their own DID
- `deleteExternalReader()` - Remove external readers from their own DID

#### **External Reader**
An address granted temporary read access to a specific DID. Access is time-limited and controlled by the DID owner with defined expiration dates.

**Reader Capabilities (while access is valid):**
- `readAttributeList()` - View attribute keys associated with the DID
- `readLinkedAddresses()` - View addresses currently linked to the DID
- `readFullDID()` - Retrieve complete DID data structure

**Access Control:** `externalReaderExpirationDate > block.timestamp`

### View Methods:

**DID Information:**
- `getUserDID(address walletAddress)` - Retrieves DID details for a wallet address (returns: UDID, validTo, updatedAt, blocked, lastUpdatedBy)
- `getLinker(address walletAddress)` - Retrieves linker information for an address (returns: uDID, joinDate, updateDate, deactivated)

**Attribute Information:**
- `getAttribute(address walletAddress, string attributeName)` - Retrieves specific attribute details (returns: value, valueType, createdAt, updatedAt, validTo, lastUpdatedBy)

**Access Verification:**
- `canRead(string uDIDToRead, address addressCanRead)` - Checks if address has permission to read DID attributes
- `externalReaderExpirationDate(string uDID, address walletAddress)` - Returns expiration timestamp for external reader access

### Write Methods:

**DID Lifecycle Management:**
- `createDID(string calldata uDID, uint256 validToData)` - Creates a new DID (WRITER_ROLE)
- `prolongateDID(string calldata uDID, uint256 newValidToData)` - Extends DID validity (WRITER_ROLE)
- `blockDID(string calldata uDID, string calldata reason)` - Blocks a DID (WRITER_ROLE)
- `unBlockDID(string calldata uDID, string calldata reason)` - Unblocks a DID (WRITER_ROLE)

**Address Management:**
- `linkAddressToDID(string calldata uDID, address addressToLink)` - Links address to DID (WRITER_ROLE or DID Owner)
- `removeLinkedAddress(string calldata uDID, address addressToDelete)` - Removes linked address (WRITER_ROLE)
- `deactivateAddressOfDID(string calldata uDID, address addressToDeactivate)` - Deactivates address (WRITER_ROLE or DID Owner)
- `activateAddressOfDID(string calldata uDID, address addressToActivate)` - Reactivates address (WRITER_ROLE or DID Owner)

**Attribute Management:**
- `addOrUpdateAttributes(ParamAttribute[] calldata paramAttributes)` - Batch add/update attributes (WRITER_ROLE)
- `deactivateDIDAttribute(string calldata uDID, string calldata attributeName)` - Deactivates attribute (WRITER_ROLE)

**External Reader Management:**
- `addOrUpdateExternalReader(string calldata uDID, address reader, uint256 expirationData)` - Adds/updates external reader (WRITER_ROLE or DID Owner)
- `deleteExternalReader(string calldata uDID, address reader)` - Removes external reader (WRITER_ROLE or DID Owner)

**Configuration:**
- `setMaxLinkedAddresses(uint256 newMax)` - Sets maximum allowed linked addresses per DID (DEFAULT_ADMIN_ROLE)

### Access-Controlled Read Methods:

Require appropriate permissions (ATTRIBUTE_READER_ROLE, DID Owner, or External Reader with valid access):

- `readFullDID(string calldata uDID)` - Reads complete DID structure
- `readAttributeList(string calldata uDID)` - Reads all attribute keys for a DID
- `readLinkedAddresses(string calldata uDID)` - Reads all addresses linked to a DID

### Events:

**DID Lifecycle Events:**
- `DIDCreated(string indexed UDID, string UDID_, address indexed createdBy)` - New DID created
- `DIDValidToDateUpdated(string indexed UDID, string UDID_, uint256 oldValidTo, uint256 indexed newValidTo, address indexed updatedBy)` - Validity date changed
- `DIDBlockStatusUpdated(string indexed UDID, string UDID_, bool indexed newBlockStatus, address indexed updatedBy, string reasonDescription)` - Block status changed

**Address Linking Events:**
- `DIDAddressLinked(string indexed UDID, string UDID_, address indexed addedAddress, address indexed updatedBy)` - Address linked to DID
- `DIDAddressDeleted(string indexed UDID, string UDID_, address indexed deletedAddress, address indexed updatedBy)` - Linked address removed
- `DIDAddressDeactivated(string indexed UDID, string UDID_, address indexed deactivatedAddress, address indexed updatedBy)` - Address deactivated
- `DIDAddressActivated(string indexed UDID, string UDID_, address indexed activatedAddress, address indexed updatedBy)` - Address reactivated

**Attribute Events:**
- `AttributeCreated(string indexed UDID, string UDID_, string indexed attributeName, address indexed createdBy)` - New attribute added
- `AttributeUpdated(string indexed UDID, string UDID_, string indexed attributeName, address indexed updatedBy)` - Attribute updated
- `AttributeDeactivated(string indexed UDID, string UDID_, string indexed attributeName, address indexed deactivatedBy)` - Attribute deactivated
- `AttributeValidToDateUpdated(string indexed UDID, string UDID_, string indexed attributeName, uint256 oldValidTo, uint256 newValidTo, address indexed updatedBy)` - Attribute validity changed

**External Reader Events:**
- `NewExternalReaderAdded` - External reader access granted
- `ExternalReaderUpdated` - External reader access updated
- `ExternalReaderDeleted` - External reader access removed

**Data Access Events:**
- `AttributeListWasRead` - Attribute list was queried
- `LinkedAddressesListWasRead` - Linked addresses list was queried
- `FullDIDWasRead(string indexed UDID, string UDID_, address whoRead)` - Full DID data was read

**Error/Debug Events:**
- `UnexpectedBehavior(string indexed UDID, string UDID_, address indexed addressToDelete, string message, address indexed addressOfBrokenLinker)` - System encountered inconsistent linkage

### Custom Errors:

```solidity
error CantRevokeLastSuperAdmin();
error CantRemoveLastLinkedAddress();
error DIDAlreadyExists(string UDID);
error DIDDoesNotExist(string UDID);
error ZeroAddressNotAllowed();
error AddressAlreadyLinkedToDID(address alreadyLinkedAddress, string uDIDLinked);
error AddressDoesNotLinkedToDID(address notLinkedAddress);
error AddressDoesNotHaveLinker(address addressWithoutLinker);
error AddressAlreadyDeactivated(address alreadyDeactivatedAddress);
error AddressAlreadyActivated(address alreadyActivatedAddress);
error NotAuthorizedForThisTransaction(address caller);
error ValidToDataMustBeInFuture(uint256 existingDateTimestamp);
error DIDIsBlocked(string UDID);
error DIDIsNotBlocked(string UDID);
error MaxLinkedAddressesExceeded(string UDID, uint256 maxAllowed);
error AddressIsNotExternalReader(string UDID, address notReaderAddress);
```

### Deployment Details:

**Arbitrum Mainnet:**
- **Chain ID:** 42161
- **Address:** [`0x25E6036178656b1329ee51696407b367D8C6ba84`](https://arbiscan.io/address/0x25E6036178656b1329ee51696407b367D8C6ba84#code)
- **Name:** StoboxDID
- **Network Status:** arbiscan.freshstatus.io
- **Documentation:** [Stobox Docs V4](https://docs.stobox.io)

**Arbitrum Sepolia Testnet:**
- **Chain ID:** 421614
- **Address:** `0xA832662d1E11F2a6cEF706cE54A993E7eeEDC440`
- **Name:** SDID
- **Network Status:** arbiscan.freshstatus.io

---

## 6. [StoboxRWAOfferingRegistry SC](https://arbiscan.io/address/0xb05cf4f652eb5475c3e4246ab976785678a1806a#code) (offering management & compliance registry)

- **Role:** Manages registration, lifecycle, and compliance validation for Security Token Offerings (STOs) tied to Real World Assets (RWAs).
  - Handles complete lifecycle: creation, activation, suspension, cancellation, termination of offerings.
  - Enforces compliance via a flexible rule engine and role-based access controls.
- **Architecture:** Built using EIP-2535 Diamond Pattern for upgradability and modular facets:
  - **DiamondCutFacet** – Handles upgrades and facet management.
  - **DiamondLoupeFacet** – Provides introspection and facet/query logic.
  - **RegistryRoleFacet** – Manages user roles and permissions (admin, tech service, offering manager, etc).
  - **OfferingGovernanceFacet** – Offering lifecycle management and governance.
  - **RegistryStorageFacet** – Reliable data storage and retrieval for offerings.
  - **RegistryRuleEngineFacet** – Enforces and manages offerings' compliance rules.
- **Interaction:**
  - Integrates with **StoboxProtocolSTV3 SC** for security token management.
  - Pulls identity data from **StoboxDID SC** via validation rules (e.g., AccreditedUSMinInvestmentRule).
  - Enables offering registration and compliance enforcement through custom rules and whitelisting.
  - Can lock and unlock security tokens based on offering state.
- **Deployed at:**
  - **Mainnet:** [`0xb05cf4f652eb5475c3e4246ab976785678a1806a`](https://arbiscan.io/address/0xb05cf4f652eb5475c3e4246ab976785678a1806a#code) ([v1.0.1 release notes](./ST4RWAOfferingRegistry-Release-Notes.md))
- **Status:** Implemented; see [release notes](./ST4RWAOfferingRegistry-Release-Notes.md) and [contract source](https://github.com/StoboxTechnologies/ST4RWAOfferingRegistry).

---

## Flow Summary

1. **StoboxRWAVaultFactory SC** creates both the **STV3Treasury SC** and **StoboxProtocolSTV3 SC**.
2. **StoboxRWAVaultFactory SC** manages the deployed protocol contracts.
3. **StoboxProtocolSTV3 SC** interacts with **Validation HasDIDRule SC** to check compliance.
4. **Validation HasDIDRule SC** queries **StoboxDID SC** to verify decentralized identities.
5. **StoboxRWAOfferingRegistry SC** manages STO lifecycle and compliance validation, integrating with the full RWA ecosystem.

---

**Integration Highlights:**
- The **StoboxRWAOfferingRegistry SC** is designed to coordinate with the full RWA ecosystem:
  - When a new offering is created, it can assign roles, configure compliance rules, require investors to meet identity/KYC requirements (via rules engine and DID checks), and manage token reservation.
  - The registry acts as a unified point of compliance, history, and role enforcement for all RWA offerings on Stobox.

---

## GitHub Public Repositories

### Smart Contracts & Blockchain Releases:
- **[Stobox-WEB3-Infrastructure](https://github.com/StoboxTechnologies/Stobox-WEB3-Infrastructure)** - Main infrastructure documentation and architecture overview
- **[Stobox_STV3_Protocol](https://github.com/StoboxTechnologies/Stobox_STV3_Protocol)** - StoboxProtocolSTV3 smart contract implementation
- **[ST4DIDSC](https://github.com/StoboxTechnologies/ST4DIDSC)** - Decentralized Identifier (DID) smart contract
- **[ST4RWAOfferingRegistry](https://github.com/StoboxTechnologies/ST4RWAOfferingRegistry)** - RWA Offering Registry implementation

### Documentation:
For detailed technical documentation about smart contracts, facets, libraries, and deployment guides, please refer to the respective repository documentation and release notes.




