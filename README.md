# Bitcoin Nexus Protocol - Smart Contract Documentation

## Overview

The Bitcoin Nexus Protocol is an enterprise-grade interoperability framework enabling secure, compliant Bitcoin-to-Stacks asset transfers. Built on Clarity's provable smart contract language, this protocol maintains Bitcoin's native security while enabling Stacks L2 smart contracts to interact programmatically with Bitcoin liquidity.

## Key Features

- **BiTC Tokenization**: 1:1 Bitcoin-backed sBTC minting with proof-of-reserves
- **Multi-Sig Oracle Network**: Federated validation from vetted Bitcoin full nodes
- **Compliance Engine**: Built-in address screening and whitelist controls
- **Institutional Safeguards**: Circuit breakers, time-locked withdrawals, cold storage hooks
- **Real-Time Audit**: Publicly verifiable reserve proofs and transaction history

## Technical Architecture

### Core Components

| Component                | Type           | Description                              |
| ------------------------ | -------------- | ---------------------------------------- |
| `Bitcoin-btc`            | Fungible Token | Wrapped Bitcoin representation on Stacks |
| `authorized-oracles`     | Map            | Oracle validation registry               |
| `recipient-whitelist`    | Map            | Compliance-approved addresses            |
| `processed-transactions` | Map            | Prevention of double-spends              |

### Protocol Parameters

| Parameter   | Variable                | Default | Description               |
| ----------- | ----------------------- | ------- | ------------------------- |
| Bridge Fee  | `bridge-fee-percentage` | 1%      | Dynamic transaction fee   |
| Max Deposit | `max-deposit-amount`    | 10 BTC  | Per-transaction limit     |
| Pause State | `is-bridge-paused`      | false   | Emergency circuit breaker |

## Workflow Overview

### Bitcoin → Stacks Transfer Process

1. **Initiation**: User submits Bitcoin transaction with OP_RETURN memo
2. **Oracle Validation**: 3/5 authorized nodes verify transaction validity
3. **Compliance Check**: Recipient address screened against OFAC lists
4. **Minting**: 1:1 sBTC minted minus protocol fees
5. **Finalization**: Transaction hash recorded to prevent replay

```clarity
(define-public (deposit-bitcoin
  (btc-tx-hash (string-ascii 64))
  (amount uint)
  (recipient principal)
)
  ;; Core minting logic with embedded compliance checks
)
```

## Security Implementation

### Multi-Layered Validation

1. **Transaction Proof**
   Schnorr signature verification (BIP-340 standard)
2. **Oracle Consensus**
   Threshold signature scheme requiring 3/5 node approval
3. **Chain Reorg Protection**
   100-block confirmation finality for Bitcoin transactions

### Emergency Protocols

```clarity
(define-public (pause-bridge)
  (begin
    (try! (check-is-bridge-owner))
    (var-set is-bridge-paused true)
    (ok true)
  )
)
```

- Immediate protocol suspension capability
- Multi-sig requirement for privileged functions
- Time-delayed parameter changes (72hr governance delay)

## Compliance Features

### Institutional Controls

- **Whitelist Management**:
  ```clarity
  (define-public (add-to-whitelist (recipient principal))
    (asserts! (is-valid-principal recipient) ERR-INVALID-RECIPIENT)
  )
  ```
- **Travel Rule Compliance**:
  Automated transaction memo field validation
- **Sanctions Screening**:
  Integration with Chainalysis/TRM APIs

## Usage Examples

### Institutional Deposit

```clarity
;; Submit BTC transaction
(invoke-contract 'bitcoin-nexus-protocol
  (deposit-bitcoin
    "a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d"
    u5000000  ;; 5 BTC
    'SP3FBR2AGK5H9QBDH3EEN6DF8EK8JY7RX8QJ5SVTE
  )
)
```

### Governance Operations

```clarity
;; Update fee structure
(invoke-contract 'bitcoin-nexus-protocol
  (update-bridge-fee u15)  ;; 1.5% fee update
)

;; Add validation oracle
(invoke-contract 'bitcoin-nexus-protocol
  (add-oracle 'SP3K8BC0PPEVCV7NZ6QSRWPQ2E9ZXC6N86NT0K2A6)
)
```

## Audit & Verification

### Proof of Reserves

```clarity
(define-read-only (get-total-locked-bitcoin)
  (var-get total-locked-bitcoin)
)
```

- Real-time reserve verification
- Monthly Merkle-proof audits
- Third-party attestation integration
