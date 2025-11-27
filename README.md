# QuillForge

A decentralized autonomous organization (DAO) smart contract built on the Stacks blockchain using Clarity. QuillForge enables community-driven treasury management through a governance token system where STX holders can participate in proposal creation, voting, and execution.

## Overview

QuillForge implements a time-locked deposit mechanism that mints governance tokens in exchange for STX deposits. Token holders gain voting power proportional to their holdings, enabling them to create and vote on treasury funding proposals. The system enforces configurable lock periods, proposal durations, and implements a simple majority voting mechanism for proposal execution.

## Features

- **STX Deposits & Withdrawals**: Users can deposit STX to receive governance tokens with time-locked vesting
- **Governance Token System**: 1:1 minting ratio between deposited STX and governance tokens
- **Proposal Management**: Create funding proposals with customizable duration and amount
- **Weighted Voting**: Vote on proposals with voting power proportional to token holdings
- **Automated Execution**: Execute approved proposals that transfer funds to designated recipients
- **Access Control**: Owner-only initialization and configurable parameters

## Contract Architecture

### State Variables

```clarity
total-supply          : uint   - Total governance tokens in circulation
minimum-deposit       : uint   - Minimum STX required for deposits (1 STX default)
lock-period           : uint   - Blocks before withdrawal allowed (~10 days)
initialized           : bool   - Contract initialization status
proposal-count        : uint   - Sequential proposal identifier counter
```

### Data Structures

#### Balances Map

```clarity
principal => uint
```

Tracks governance token balances for each participant.

#### Deposits Map

```clarity
principal => {
  amount: uint,
  lock-until: uint,
  last-reward-block: uint
}
```

Records deposit details including lock period and reward tracking.

#### Proposals Map

```clarity
uint => {
  proposer: principal,
  description: string-ascii 256,
  amount: uint,
  target: principal,
  expires-at: uint,
  executed: bool,
  yes-votes: uint,
  no-votes: uint
}
```

Stores proposal metadata and voting results.

#### Votes Map

```clarity
{proposal-id: uint, voter: principal} => bool
```

Prevents double voting by tracking individual vote records.

## Data Flow

### Deposit Flow

```
User → deposit(amount)
  ├─ Validate: amount >= minimum-deposit
  ├─ Transfer: STX from user to contract
  ├─ Record: deposit info with lock period
  └─ Mint: governance tokens 1:1 to user
```

### Proposal Flow

```
Token Holder → create-proposal(description, amount, target, duration)
  ├─ Validate: holder has tokens
  ├─ Validate: duration bounds (1-14 days)
  ├─ Create: new proposal with unique ID
  └─ Return: proposal-id

Token Holders → vote(proposal-id, vote-for)
  ├─ Validate: voter has tokens
  ├─ Validate: not expired
  ├─ Validate: no double voting
  └─ Record: weighted vote

Anyone → execute-proposal(proposal-id)
  ├─ Validate: proposal expired (voting ended)
  ├─ Validate: yes-votes > no-votes
  ├─ Validate: sufficient contract balance
  ├─ Transfer: funds to target
  └─ Mark: proposal as executed
```

### Withdrawal Flow

```
User → withdraw(amount)
  ├─ Validate: lock period elapsed
  ├─ Validate: sufficient token balance
  ├─ Burn: governance tokens
  └─ Transfer: STX back to user
```

## Security Features

- **Time Locks**: Prevents immediate withdrawal, ensuring commitment
- **Single Vote Prevention**: Mapping structure ensures one vote per user per proposal
- **Proposal Expiration**: Time-bounded voting windows prevent stale proposals
- **Execution Validation**: Multi-check validation before fund transfers
- **Balance Checks**: Comprehensive balance validation across all operations
- **Authorization Guards**: Token ownership required for voting and proposal creation

## Error Codes

| Code | Error | Description |
|------|-------|-------------|
| u100 | `err-owner-only` | Action restricted to contract owner |
| u101 | `err-not-initialized` | Contract must be initialized first |
| u102 | `err-insufficient-balance` | Insufficient token or STX balance |
| u103 | `err-already-initialized` | Contract already initialized |
| u104 | `err-unauthorized` | Caller lacks required permissions |
| u105 | `err-proposal-not-found` | Invalid proposal ID |
| u106 | `err-proposal-expired` | Proposal voting period ended/not ended |
| u107 | `err-already-voted` | User already voted on this proposal |
| u108 | `err-below-minimum` | Deposit below minimum threshold |
| u110 | `err-locked-period` | Withdrawal lock period not elapsed |
| u111 | `err-transfer-failed` | STX transfer failed |
| u112 | `err-invalid-duration` | Proposal duration out of bounds |
| u113 | `err-zero-amount` | Amount must be greater than zero |
| u114 | `err-invalid-target` | Invalid recipient address |
| u115 | `err-invalid-description` | Empty proposal description |
| u116 | `err-invalid-proposal-id` | Proposal ID out of range |
| u117 | `err-invalid-vote` | Invalid vote value |

## Public Functions

### `initialize()`

Initializes the contract. Must be called by contract owner before any other operations.

**Returns**: `(response bool uint)`

---

### `deposit(amount: uint)`

Deposits STX and receives governance tokens.

**Parameters**:

- `amount`: STX amount in microSTX (minimum 1,000,000 µSTX)

**Returns**: `(response bool uint)`

---

### `withdraw(amount: uint)`

Burns governance tokens and withdraws STX after lock period.

**Parameters**:

- `amount`: Governance token amount to burn

**Returns**: `(response bool uint)`

---

### `create-proposal(description: string-ascii, amount: uint, target: principal, duration: uint)`

Creates a new funding proposal.

**Parameters**:

- `description`: Proposal description (max 256 chars)
- `amount`: Funding amount in microSTX
- `target`: Recipient address
- `duration`: Voting period in blocks (144-20,160 blocks / ~1-14 days)

**Returns**: `(response uint uint)` - Returns proposal ID on success

---

### `vote(proposal-id: uint, vote-for: bool)`

Casts a weighted vote on an active proposal.

**Parameters**:

- `proposal-id`: Target proposal ID
- `vote-for`: true for yes, false for no

**Returns**: `(response bool uint)`

---

### `execute-proposal(proposal-id: uint)`

Executes an approved proposal after voting period ends.

**Parameters**:

- `proposal-id`: Proposal to execute

**Returns**: `(response bool uint)`

## Read-Only Functions

### `get-balance(account: principal)`

Returns governance token balance for an account.

### `get-total-supply()`

Returns total governance tokens in circulation.

### `get-proposal(proposal-id: uint)`

Returns proposal details by ID.

### `get-deposit-info(account: principal)`

Returns deposit information for an account.

### `get-vote(proposal-id: uint, voter: principal)`

Returns how a user voted on a specific proposal.

## Development Setup

### Prerequisites

- [Clarinet](https://github.com/hirosystems/clarinet) >= 2.0
- Node.js >= 18.x
- npm or yarn

### Installation

```bash
# Clone repository
git clone https://github.com/mwangi-otieno/quill-forge.git
cd quill-forge

# Install dependencies
npm install
```

### Testing

```bash
# Check contract syntax
clarinet check

# Run test suite
npm test

# Or use Clarinet tasks
clarinet test
```

### Deployment

```bash
# Deploy to devnet
clarinet integrate

# Deploy to testnet/mainnet
clarinet publish
```

## Configuration

Contract parameters can be adjusted before deployment:

- `minimum-deposit`: Default 1,000,000 µSTX (1 STX)
- `lock-period`: Default 1,440 blocks (~10 days)
- `minimum-duration`: Default 144 blocks (~1 day)
- `maximum-duration`: Default 20,160 blocks (~14 days)

## Use Cases

- **Community Treasuries**: Manage shared funds democratically
- **Grant Programs**: Vote on funding allocations for projects
- **Protocol Governance**: Coordinate protocol parameter changes
- **Investment DAOs**: Pool capital and vote on investments
- **Development Funding**: Community-driven project sponsorship

## Limitations

- Simple majority voting (no quorum requirements)
- No vote delegation or proxy voting
- Fixed 1:1 token minting ratio
- No proposal cancellation mechanism
- No vote changing after submission

## License

MIT

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## Security

This contract has not been audited. Use at your own risk. For production deployments, a professional security audit is strongly recommended.
