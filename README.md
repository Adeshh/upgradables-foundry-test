# Upgradeable Proxy Foundry Project

A Foundry project demonstrating how to implement and upgrade UUPS (Universal Upgradeable Proxy Standard) proxies using OpenZeppelin's upgradeable contracts.

## Overview

This project demonstrates:
- Deploying an upgradeable proxy contract using the UUPS pattern
- Upgrading the implementation contract while preserving storage
- Testing upgrade functionality with Foundry

## Architecture

### Contracts

- **BoxV1** (`src/BoxV1.sol`): Initial implementation contract
  - Inherits from `UUPSUpgradeable` and `OwnableUpgradeable`
  - Implements `initialize()` function for initialization
  - Contains a `number` state variable and `getNumber()` function
  - Version 1

- **BoxV2** (`src/BoxV2.sol`): Upgraded implementation contract
  - Inherits from `UUPSUpgradeable`
  - Adds `setNumber()` function (new functionality)
  - Preserves storage layout from BoxV1
  - Version 2

### Proxy Pattern

The project uses the **UUPS (Universal Upgradeable Proxy Standard)** pattern:
- The proxy contract (`ERC1967Proxy`) stores the implementation address
- All calls to the proxy are delegated to the implementation contract
- Upgrades are performed by calling `upgradeToAndCall()` on the proxy
- Storage is preserved across upgrades

## Prerequisites

- [Foundry](https://book.getfoundry.sh/getting-started/installation)
- Solidity ^0.8.18

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd upgradable-proxy-foundry
```

2. Install dependencies:
```bash
forge install
```

## Usage

### Build

Compile the contracts:
```bash
forge build
```

### Test

Run the test suite:
```bash
forge test
```

The test suite includes:
- Verifying the proxy is initialized with BoxV1
- Testing the upgrade to BoxV2
- Verifying storage persistence after upgrade

### Deploy

Deploy the proxy and implementation:

```bash
forge script script/DeployBox.s.sol:DeployBox --rpc-url <your_rpc_url> --private-key <your_private_key> --broadcast
```

### Upgrade

Upgrade the proxy to a new implementation:

```bash
forge script script/UpgradeBox.s.sol:UpgradeBox --rpc-url <your_rpc_url> --private-key <your_private_key> --broadcast
```

## Project Structure

```
.
├── src/
│   ├── BoxV1.sol          # Initial implementation
│   └── BoxV2.sol          # Upgraded implementation
├── script/
│   ├── DeployBox.s.sol    # Deployment script
│   └── UpgradeBox.s.sol   # Upgrade script
├── test/
│   └── DeployAndUpgradeTest.t.sol  # Test suite
├── foundry.toml           # Foundry configuration
└── README.md
```

## How Upgrades Work

1. **Initial Deployment**: 
   - Deploy `BoxV1` implementation contract
   - Deploy `ERC1967Proxy` pointing to `BoxV1`
   - Call `initialize()` on the proxy to set up initial state

2. **Upgrade Process**:
   - Deploy new `BoxV2` implementation contract
   - Call `upgradeToAndCall()` on the proxy with the new implementation address
   - The proxy's storage slot is updated to point to `BoxV2`
   - All existing storage (like `number`) is preserved

3. **Storage Layout**:
   - Both `BoxV1` and `BoxV2` must maintain the same storage layout
   - New state variables can only be added at the end
   - Never remove or reorder existing state variables

## Key Concepts

### UUPS Pattern
- Upgrade logic is in the implementation contract, not the proxy
- More gas-efficient than transparent proxies
- Requires careful access control via `_authorizeUpgrade()`

### Initialization
- Use `initialize()` instead of constructors for upgradeable contracts
- Call `_disableInitializers()` in the constructor to prevent initialization on implementation
- Only initialize through the proxy

### Storage Layout
- Storage slots must remain consistent across upgrades
- Use `@custom:storage-location` annotations for complex storage
- Follow OpenZeppelin's upgradeable contract guidelines

## Security Considerations

⚠️ **Important**: This is a demonstration project. For production use:
- Implement proper access control in `_authorizeUpgrade()`
- Use OpenZeppelin's upgrade plugin for validation
- Thoroughly test upgrades on testnets before mainnet

## Dependencies

- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts)
- [OpenZeppelin Contracts Upgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable)
- [Foundry](https://github.com/foundry-rs/foundry)

## Resources

- [OpenZeppelin Upgradeable Contracts Docs](https://docs.openzeppelin.com/upgrades-plugins/1.x/)
- [Foundry Book](https://book.getfoundry.sh/)
- [EIP-1967: Proxy Storage Slots](https://eips.ethereum.org/EIPS/eip-1967)
- [EIP-1822: Universal Upgradeable Proxy Standard](https://eips.ethereum.org/EIPS/eip-1822)
