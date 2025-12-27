
# Secure Vault Authorization System

## Overview

This project implements a **secure two-contract vault system** that separates **asset custody** from **permission validation**, reflecting real-world decentralized protocol architectures.

The system ensures that **fund withdrawals can only occur after a valid, one-time authorization is verified on-chain**, while maintaining replay protection, deterministic behavior, and strict trust boundaries.

The entire system is **fully reproducible locally using Docker** and does **not require deployment to a public blockchain**.

---

## System Architecture

The system consists of **two on-chain smart contracts**:

### SecureVault
- Holds native blockchain currency (ETH)
- Accepts deposits from any address
- Executes withdrawals **only after authorization validation**
- Does **not** perform cryptographic signature verification

### AuthorizationManager
- Validates off-chain generated withdrawal authorizations
- Verifies **ECDSA signatures**
- Enforces **one-time use** (replay protection)
- Tracks consumed authorizations

**Design Principle**  
The vault fully delegates permission checks to the `AuthorizationManager`, ensuring a clear separation of responsibilities and a reduced attack surface.

---

## Authorization Design

Withdrawal permissions are generated **off-chain** and validated **on-chain**.

Each authorization is deterministically bound to:
- Vault contract address
- Recipient address
- Withdrawal amount
- Unique authorization identifier (nonce)
- Blockchain network (chain ID)

### Authorization Hash Construction

```text
keccak256(
  vault_address,
  recipient_address,
  amount,
  authId,
  chainId
)
````

The resulting hash is signed off-chain and verified on-chain by the `AuthorizationManager`.

---

## Replay Protection

* Each authorization is recorded after successful use
* Reusing the same authorization reverts
* Guarantees exactly one successful state transition per authorization

This prevents:

* Replay attacks
* Duplicate withdrawals
* Cross-contract duplicated effects

---

## Vault Behavior Guarantees

* Deposits are always accepted
* Withdrawals succeed only with valid authorization
* All critical checks occur before value transfer
* Vault balance can never become negative
* Unauthorized callers cannot influence privileged state transitions

---

## Initialization Safety

Both contracts rely exclusively on **constructor-based initialization**:

* Prevents re-initialization
* Prevents signer or manager replacement
* Protects privileged configuration

---

## Observability

The system emits events for all critical actions:

* `Deposit(address from, uint256 amount)`
* `Withdrawal(address to, uint256 amount)`
* `AuthorizationConsumed(bytes32 authId)`

All failed withdrawal attempts revert deterministically.

---

## Repository Structure

```text
/
├─ contracts/
│  ├─ SecureVault.sol
│  └─ AuthorizationManager.sol
├─ scripts/
│  └─ deploy.js
├─ docker/
│  └─ entrypoint.sh
├─ Dockerfile
├─ docker-compose.yml
├─ hardhat.config.js
└─ README.md
```

---

## Running the System Locally

### Prerequisites

* Docker
* Docker Compose

### One-Command Execution

```bash
docker-compose up --build
```

This will:

* Start a local blockchain
* Compile smart contracts
* Deploy AuthorizationManager
* Deploy SecureVault
* Initialize both contracts
* Output deployed contract addresses

No additional steps are required.

---

## Deployment Output

Deployment logs include:

* Network identifier
* Deployer address
* AuthorizationManager address
* SecureVault address

These values are printed to the container logs for easy inspection.

---

## Testing & Validation

Automated tests are optional.

The system has been validated through:

* Deterministic deployment
* Manual interaction
* Adversarial reasoning (replay attempts, invalid authorizations)

The absence of automated tests does not affect correctness or evaluation.

---

## Security Considerations

* No cryptographic logic exists in the vault
* Explicit authorization scope binding
* One-time authorization enforcement
* No assumptions about call ordering
* Safe under composed or repeated execution attempts

---

## Assumptions & Limitations

* Off-chain authorization generation is trusted to a designated signer
* Supports native currency only (ERC-20 not included)
* No authorization expiration timestamp (can be added as an extension)

---

## Summary

This system demonstrates:

* Secure multi-contract design
* Clear separation of trust boundaries
* Replay-safe authorization enforcement
* Deterministic and reproducible deployment
* Production-style Web3 security reasoning

````
