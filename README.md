# Multi-Signature Wallet

A secure, gas-optimized multi-signature wallet smart contract built with Solidity. This wallet requires multiple owner signatures before executing any transaction, providing enhanced security for managing shared funds.

## 🔥 Features

- **Multi-signature security**: Configurable M-of-N signature requirements
- **Owner management**: Add/remove owners through consensus
- **Transaction queue**: Submit, confirm, and execute transactions in stages
- **Revocation support**: Owners can revoke confirmations before execution
- **Gas optimized**: Efficient storage patterns and minimal gas usage
- **Event logging**: Complete audit trail of all operations
- **Flexible requirements**: Change signature thresholds dynamically

## 🚀 Quick Start

### Prerequisites

- Node.js >= 14.0.0
- npm or yarn
- Hardhat

### Installation

1. Clone the repository:
```bash
git clone <your-repo-url>
cd multisig-wallet
```

2. Install dependencies:
```bash
npm install
```

3. Install Hardhat and plugins:
```bash
npm install --save-dev hardhat @nomiclabs/hardhat-waffle @nomiclabs/hardhat-ethers ethereum-waffle chai
```

### Compilation

```bash
npx hardhat compile
```

### Testing

```bash
npx hardhat test
```

### Deployment

1. Configure your network in `hardhat.config.js`
2. Update the deployment script with your desired parameters
3. Deploy:

```bash
# Local development
npx hardhat run scripts/deploy.js --network localhost

# Testnet (e.g., Goerli)
npx hardhat run scripts/deploy.js --network goerli

# Mainnet
npx hardhat run scripts/deploy.js --network mainnet
```

## 📋 Usage Guide

### Basic Workflow

1. **Submit Transaction**: Any owner can propose a transaction
2. **Confirm Transaction**: Other owners confirm the transaction
3. **Execute Transaction**: Once enough confirmations are reached, execute

### Code Examples

#### Deploy a 2-of-3 MultiSig Wallet

```javascript
const owners = [
  "0x1234567890123456789012345678901234567890",
  "0x2345678901234567890123456789012345678901",
  "0x3456789012345678901234567890123456789012"
];
const requiredConfirmations = 2;

const MultiSigWallet = await ethers.getContractFactory("MultiSigWallet");
const wallet = await MultiSigWallet.deploy(owners, requiredConfirmations);
```

#### Submit a Transaction

```javascript
const recipient = "0x742d35Cc6634C0532925a3b8D49d2C6";
const amount = ethers.utils.parseEther("1.0");
const data = "0x"; // Empty for simple ETH transfer

await wallet.connect(owner1).submitTransaction(recipient, amount, data);
```

#### Confirm and Execute

```javascript
const txIndex = 0;

// Owners confirm
await wallet.connect(owner1).confirmTransaction(txIndex);
await wallet.connect(owner2).confirmTransaction(txIndex);

// Execute when enough confirmations
await wallet.connect(owner1).executeTransaction(txIndex);
```

#### Monitor Pending Transactions

```javascript
// Get all pending transactions
const pendingTxs = await wallet.getPendingTransactions();

// Check transaction details
for (let txIndex of pendingTxs) {
  const tx = await wallet.getTransaction(txIndex);
  console.log(`Transaction ${txIndex}:`, {
    to: tx.to,
    value: ethers.utils.formatEther(tx.value),
    confirmations: tx.numConfirmations.toString()
  });
}
```

## 🔧 Contract Interface

### Main Functions

| Function | Description | Access |
|----------|-------------|--------|
| `submitTransaction(address _to, uint256 _value, bytes _data)` | Submit a new transaction proposal | Owners only |
| `confirmTransaction(uint256 _txIndex)` | Confirm a pending transaction | Owners only |
| `executeTransaction(uint256 _txIndex)` | Execute a confirmed transaction | Owners only |
| `revokeConfirmation(uint256 _txIndex)` | Revoke your confirmation | Owners only |

### View Functions

| Function | Description | Returns |
|----------|-------------|---------|
| `getOwners()` | Get all wallet owners | `address[]` |
| `getTransactionCount()` | Get total transaction count | `uint256` |
| `getTransaction(uint256 _txIndex)` | Get transaction details | `(address, uint256, bytes, bool, uint256)` |
| `isConfirmed(uint256 _txIndex, address _owner)` | Check if owner confirmed transaction | `bool` |
| `getBalance()` | Get wallet ETH balance | `uint256` |

### Events

```solidity
event Deposit(address indexed sender, uint256 amount, uint256 balance);
event SubmitTransaction(address indexed owner, uint256 indexed txIndex, address indexed to, uint256 value, bytes data);
event ConfirmTransaction(address indexed owner, uint256 indexed txIndex);
event ExecuteTransaction(address indexed owner, uint256 indexed txIndex);
```

## 🛡️ Security Features

### Access Control
- Only designated owners can interact with the wallet
- All critical functions are protected by `onlyOwner` modifier
- Transaction execution requires minimum confirmation threshold

### Reentrancy Protection
- State changes occur before external calls
- Failed transactions revert completely
- No partial state updates

### Input Validation
- Comprehensive parameter validation
- Zero address checks
- Duplicate owner prevention
- Valid confirmation requirement checks

## 📊 Gas Optimization

### Storage Efficiency
- Packed structs to minimize storage slots
- Efficient mapping usage
- Minimal redundant data

### Function Optimization
- Batch operations support
- Early returns for gas savings
- Optimized loops and conditions

### Typical Gas Costs
- Deploy (3 owners, 2 confirmations): ~2,500,000 gas
- Submit transaction: ~100,000 gas
- Confirm transaction: ~80,000 gas
- Execute transaction: ~50,000 + destination gas

## 🧪 Testing

The project includes comprehensive tests covering:

- ✅ Deployment scenarios
- ✅ Transaction submission
- ✅ Confirmation workflows
- ✅ Execution conditions
- ✅ Access control
- ✅ Edge cases
- ✅ Security vulnerabilities

Run tests with:
```bash
npx hardhat test
```

For gas reporting:
```bash
npx hardhat test --gas-reporter
```

## 🌐 Network Configuration

### Hardhat Config Example

```javascript
require("@nomiclabs/hardhat-waffle");

module.exports = {
  solidity: {
    version: "0.8.19",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    localhost: {
      url: "http://127.0.0.1:8545"
    },
    goerli: {
      url: `https://goerli.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [PRIVATE_KEY]
    },
    mainnet: {
      url: `https://mainnet.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [PRIVATE_KEY]
    }
  }
};
```

## 🏛️ Common Use Cases

### 1. DAO Treasury Management
```javascript
// 5-of-7 governance council
const daoWallet = await MultiSigWallet.deploy(
  [council1, council2, council3, council4, council5, council6, council7],
  5
);
```

### 2. Business Partnership
```javascript
// 2-of-3 with external arbitrator
const businessWallet = await MultiSigWallet.deploy(
  [partner1, partner2, arbitrator],
  2
);
```

### 3. Investment Fund
```javascript
// 3-of-5 investment committee
const fundWallet = await MultiSigWallet.deploy(
  [gp1, gp2, lp1, advisor1, advisor2],
  3
);
```

## 🔍 Verification

### Etherscan Verification

```bash
npx hardhat verify --network mainnet CONTRACT_ADDRESS "CONSTRUCTOR_ARG1" "CONSTRUCTOR_ARG2"
```

### Manual Verification Steps

1. Deploy contract with known parameters
2. Verify owner addresses match
3. Confirm required confirmations setting
4. Test transaction submission and confirmation
5. Verify event emissions

## 🚨 Security Considerations

### Best Practices

1. **Owner Key Management**
   - Use hardware wallets for owner accounts
   - Store keys in different locations/devices
   - Never share private keys

2. **Transaction Verification**
   - Always verify transaction details before confirming
   - Check recipient addresses carefully
   - Validate transaction data for contract calls

3. **Regular Audits**
   - Monitor pending transactions regularly
   - Review owner list periodically
   - Check for unexpected transactions

### Known Limitations

- No time-locks on transactions (can be added in extensions)
- No spending limits (by design for flexibility)
- Owner changes require careful coordination

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

## 📞 Support

- GitHub Issues: [Create an issue](https://github.com/your-repo/issues)
- Documentation: [Wiki](https://github.com/your-repo/wiki)
- Security Issues: Please report privately

## 🗺️ Roadmap

- [ ] Time-locked transactions
- [ ] Spending limits and budgets
- [ ] Mobile app integration
- [ ] Advanced analytics dashboard
- [ ] Multi-token support enhancements
- [ ] Gasless transactions (meta-transactions)

---

**⚠️ Disclaimer**: This smart contract has been developed for educational and demonstration purposes. While it follows security best practices, please conduct thorough testing and consider professional audit before using in production with significant funds.# Multi-Signature Wallet

A secure, gas-optimized multi-signature wallet smart contract built with Solidity. This wallet requires multiple owner signatures before executing any transaction, providing enhanced security for managing shared funds.

## 🔥 Features

- **Multi-signature security**: Configurable M-of-N signature requirements
- **Owner management**: Add/remove owners through consensus
- **Transaction queue**: Submit, confirm, and execute transactions in stages
- **Revocation support**: Owners can revoke confirmations before execution
- **Gas optimized**: Efficient storage patterns and minimal gas usage
- **Event logging**: Complete audit trail of all operations
- **Flexible requirements**: Change signature thresholds dynamically

## 🚀 Quick Start

### Prerequisites

- Node.js >= 14.0.0
- npm or yarn
- Hardhat

### Installation

1. Clone the repository:
```bash
git clone <your-repo-url>
cd multisig-wallet
```

2. Install dependencies:
```bash
npm install
```

3. Install Hardhat and plugins:
```bash
npm install --save-dev hardhat @nomiclabs/hardhat-waffle @nomiclabs/hardhat-ethers ethereum-waffle chai
```

### Compilation

```bash
npx hardhat compile
```

### Testing

```bash
npx hardhat test
```

### Deployment

1. Configure your network in `hardhat.config.js`
2. Update the deployment script with your desired parameters
3. Deploy:

```bash
# Local development
npx hardhat run scripts/deploy.js --network localhost

# Testnet (e.g., Goerli)
npx hardhat run scripts/deploy.js --network goerli

# Mainnet
npx hardhat run scripts/deploy.js --network mainnet
```

## 📋 Usage Guide

### Basic Workflow

1. **Submit Transaction**: Any owner can propose a transaction
2. **Confirm Transaction**: Other owners confirm the transaction
3. **Execute Transaction**: Once enough confirmations are reached, execute

### Code Examples

#### Deploy a 2-of-3 MultiSig Wallet

```javascript
const owners = [
  "0x1234567890123456789012345678901234567890",
  "0x2345678901234567890123456789012345678901",
  "0x3456789012345678901234567890123456789012"
];
const requiredConfirmations = 2;

const MultiSigWallet = await ethers.getContractFactory("MultiSigWallet");
const wallet = await MultiSigWallet.deploy(owners, requiredConfirmations);
```

#### Submit a Transaction

```javascript
const recipient = "0x742d35Cc6634C0532925a3b8D49d2C6";
const amount = ethers.utils.parseEther("1.0");
const data = "0x"; // Empty for simple ETH transfer

await wallet.connect(owner1).submitTransaction(recipient, amount, data);
```

#### Confirm and Execute

```javascript
const txIndex = 0;

// Owners confirm
await wallet.connect(owner1).confirmTransaction(txIndex);
await wallet.connect(owner2).confirmTransaction(txIndex);

// Execute when enough confirmations
await wallet.connect(owner1).executeTransaction(txIndex);
```

#### Monitor Pending Transactions

```javascript
// Get all pending transactions
const pendingTxs = await wallet.getPendingTransactions();

// Check transaction details
for (let txIndex of pendingTxs) {
  const tx = await wallet.getTransaction(txIndex);
  console.log(`Transaction ${txIndex}:`, {
    to: tx.to,
    value: ethers.utils.formatEther(tx.value),
    confirmations: tx.numConfirmations.toString()
  });
}
```

## 🔧 Contract Interface

### Main Functions

| Function | Description | Access |
|----------|-------------|--------|
| `submitTransaction(address _to, uint256 _value, bytes _data)` | Submit a new transaction proposal | Owners only |
| `confirmTransaction(uint256 _txIndex)` | Confirm a pending transaction | Owners only |
| `executeTransaction(uint256 _txIndex)` | Execute a confirmed transaction | Owners only |
| `revokeConfirmation(uint256 _txIndex)` | Revoke your confirmation | Owners only |

### View Functions

| Function | Description | Returns |
|----------|-------------|---------|
| `getOwners()` | Get all wallet owners | `address[]` |
| `getTransactionCount()` | Get total transaction count | `uint256` |
| `getTransaction(uint256 _txIndex)` | Get transaction details | `(address, uint256, bytes, bool, uint256)` |
| `isConfirmed(uint256 _txIndex, address _owner)` | Check if owner confirmed transaction | `bool` |
| `getBalance()` | Get wallet ETH balance | `uint256` |

### Events

```solidity
event Deposit(address indexed sender, uint256 amount, uint256 balance);
event SubmitTransaction(address indexed owner, uint256 indexed txIndex, address indexed to, uint256 value, bytes data);
event ConfirmTransaction(address indexed owner, uint256 indexed txIndex);
event ExecuteTransaction(address indexed owner, uint256 indexed txIndex);
```

## 🛡️ Security Features

### Access Control
- Only designated owners can interact with the wallet
- All critical functions are protected by `onlyOwner` modifier
- Transaction execution requires minimum confirmation threshold

### Reentrancy Protection
- State changes occur before external calls
- Failed transactions revert completely
- No partial state updates

### Input Validation
- Comprehensive parameter validation
- Zero address checks
- Duplicate owner prevention
- Valid confirmation requirement checks

## 📊 Gas Optimization

### Storage Efficiency
- Packed structs to minimize storage slots
- Efficient mapping usage
- Minimal redundant data

### Function Optimization
- Batch operations support
- Early returns for gas savings
- Optimized loops and conditions

### Typical Gas Costs
- Deploy (3 owners, 2 confirmations): ~2,500,000 gas
- Submit transaction: ~100,000 gas
- Confirm transaction: ~80,000 gas
- Execute transaction: ~50,000 + destination gas

## 🧪 Testing

The project includes comprehensive tests covering:

- ✅ Deployment scenarios
- ✅ Transaction submission
- ✅ Confirmation workflows
- ✅ Execution conditions
- ✅ Access control
- ✅ Edge cases
- ✅ Security vulnerabilities

Run tests with:
```bash
npx hardhat test
```

For gas reporting:
```bash
npx hardhat test --gas-reporter
```

## 🌐 Network Configuration

### Hardhat Config Example

```javascript
require("@nomiclabs/hardhat-waffle");

module.exports = {
  solidity: {
    version: "0.8.19",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    localhost: {
      url: "http://127.0.0.1:8545"
    },
    goerli: {
      url: `https://goerli.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [PRIVATE_KEY]
    },
    mainnet: {
      url: `https://mainnet.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [PRIVATE_KEY]
    }
  }
};
```

## 🏛️ Common Use Cases

### 1. DAO Treasury Management
```javascript
// 5-of-7 governance council
const daoWallet = await MultiSigWallet.deploy(
  [council1, council2, council3, council4, council5, council6, council7],
  5
);
```

### 2. Business Partnership
```javascript
// 2-of-3 with external arbitrator
const businessWallet = await MultiSigWallet.deploy(
  [partner1, partner2, arbitrator],
  2
);
```

### 3. Investment Fund
```javascript
// 3-of-5 investment committee
const fundWallet = await MultiSigWallet.deploy(
  [gp1, gp2, lp1, advisor1, advisor2],
  3
);
```

## 🔍 Verification

### Etherscan Verification

```bash
npx hardhat verify --network mainnet CONTRACT_ADDRESS "CONSTRUCTOR_ARG1" "CONSTRUCTOR_ARG2"
```

### Manual Verification Steps

1. Deploy contract with known parameters
2. Verify owner addresses match
3. Confirm required confirmations setting
4. Test transaction submission and confirmation
5. Verify event emissions

## 🚨 Security Considerations

### Best Practices

1. **Owner Key Management**
   - Use hardware wallets for owner accounts
   - Store keys in different locations/devices
   - Never share private keys

2. **Transaction Verification**
   - Always verify transaction details before confirming
   - Check recipient addresses carefully
   - Validate transaction data for contract calls

3. **Regular Audits**
   - Monitor pending transactions regularly
   - Review owner list periodically
   - Check for unexpected transactions

### Known Limitations

- No time-locks on transactions (can be added in extensions)
- No spending limits (by design for flexibility)
- Owner changes require careful coordination

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

## 📞 Support

- GitHub Issues: [Create an issue](https://github.com/your-repo/issues)
- Documentation: [Wiki](https://github.com/your-repo/wiki)
- Security Issues: Please report privately

## 🗺️ Roadmap

- [ ] Time-locked transactions
- [ ] Spending limits and budgets
- [ ] Mobile app integration
- [ ] Advanced analytics dashboard
- [ ] Multi-token support enhancements
- [ ] Gasless transactions (meta-transactions)

---

**⚠️ Disclaimer**: This smart contract has been developed for educational and demonstration purposes. While it follows security best practices, please conduct thorough testing and consider professional audit before using in production with significant funds.
 ![image](https://github.com/user-attachments/assets/52ffb330-f8fe-446b-a879-c556236308f2)
