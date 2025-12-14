# 🎰 Provably Fair Smart Contract Lottery

A decentralized, automated, and tamper-proof lottery system built with **Foundry**, **Chainlink VRF**, and **Chainlink Automation**.

## 📝 About The Project

This project implements a robust smart contract lottery where users can buy tickets with ETH. After a set period of time, the contract automatically draws a winner who receives the entire pot.

Unlike traditional lotteries, this system is **provably fair** and **fully autonomous**:

- **True Randomness:** Uses **Chainlink VRF (Verifiable Random Function)** to select a winner. This ensures the random number is generated off-chain and verified on-chain, making it mathematically impossible for miners or the contract owner to manipulate the result.
- **Automation:** Uses **Chainlink Automation** (formerly Keepers) to trigger the lottery draw automatically once the time interval has passed. No manual intervention is required.

## 🧩 Key Features

- **Immutable Entrance Fee:** Gas-efficient fee structure set at deployment
- **Dynamic Player Tracking:** Scalable array data structure to handle unlimited participants
- **State Management:** Secure `OPEN` and `CALCULATING` states to prevent race conditions during the random number generation phase
- **Secure Transfers:** Uses the Checks-Effects-Interactions (CEI) pattern to prevent reentrancy attacks
- **Gas Optimized:** Custom Errors (e.g., `Raffle__NotEnoughEthSent`) used instead of expensive string revert messages
- **Automated Infrastructure:** Custom scripts to programmatically create subscriptions, fund them with LINK, and add consumers—handling both local (Mock) and testnet environments automatically

## 🛠 Tech Stack

- **Language:** Solidity ^0.8.19
- **Framework:** Foundry (Forge, Cast, Anvil, Chisel)
- **Oracles:** Chainlink VRF v2.5, Chainlink Automation
- **Libraries:** Solmate, Foundry-DevOps

## 🚀 Getting Started

### Prerequisites

- **Foundry:** You must have Foundry installed.
  ```bash
  curl -L https://foundry.paradigm.xyz | bash
  foundryup
  ```
- **Git:** To clone the repository.

### Installation

1. **Clone the repo**
   ```bash
   git clone https://github.com/your-username/foundry-smart-contract-lottery.git
   cd foundry-smart-contract-lottery
   ```

2. **Install Dependencies**
   
   This project relies on `chainlink-brownie-contracts`, `solmate`, and `foundry-devops`.
   ```bash
   forge install smartcontractkit/chainlink-brownie-contracts@1.1.1 --no-commit
   forge install transmissions11/solmate@v6 --no-commit
   forge install Cyfrin/foundry-devops --no-commit
   ```

3. **Build**
   
   Compile the contracts to ensure everything is set up correctly.
   ```bash
   forge build
   ```

## 🧪 Testing

This project uses a comprehensive testing strategy including Unit, Integration, Fuzz, and Forked tests.

### Unit & Integration Tests (Local)

Run tests on the local Anvil chain using Mocks for VRF and LINK.

```bash
forge test
```

### Coverage Report

See which parts of the code are covered by tests.

```bash
forge coverage
```

### Forked Testnet Testing

Test against a simulation of the Sepolia testnet to verify integration with real Chainlink contracts.

*Note: Requires `SEPOLIA_RPC_URL` in your `.env` file.*

```bash
forge test --fork-url $SEPOLIA_RPC_URL
```



## 📁 Project Structure

```
foundry-smart-contract-lottery/
├── script/                     # Deployment & interaction scripts
│   ├── DeployRaffle.s.sol     # Main deployment script
│   ├── HelperConfig.s.sol     # Network configuration manager
│   └── Interactions.s.sol     # Subscription management scripts
├── src/                        # Smart contracts
│   └── Raffle.sol             # Main lottery contract
├── test/                       # Test files
│   ├── integration/           # Integration tests
│   ├── mocks/                 # Mock contracts for testing
│   │   └── LinkToken.sol      # Mock LINK token
│   └── unit/                  # Unit tests
│       └── RaffleTest.t.sol   # Raffle contract tests
└── README.md                  # This file
```

## 🏗 Architecture Details

### [`src/Raffle.sol`](./src/Raffle.sol)

The core lottery contract containing all the main logic.

- **`enterRaffle()`**: Allows users to enter. Checks for ETH value and OPEN state.
- **`checkUpkeep()`**: A `view` function called by Chainlink nodes to verify if the lottery is ready to run (Time passed + Has Players + Has Balance).
- **`performUpkeep()`**: The trigger function. Validates `checkUpkeep`, locks the lottery state to `CALCULATING`, and requests a random number from the VRF Coordinator.
- **`fulfillRandomWords()`**: The callback function called by the VRF Coordinator. It picks the winner using `random % players.length`, transfers the balance, resets the array, and reopens the lottery.

### [`script/HelperConfig.s.sol`](./script/HelperConfig.s.sol)

Acts as a configuration manager. It returns the correct addresses (VRF Coordinator, Link Token, Key Hash) based on the active chain ID (Sepolia or Anvil).

### [`script/Interactions.s.sol`](./script/Interactions.s.sol)

Handles the programmatic management of Chainlink subscriptions.

- **`CreateSubscription`**: Creates a new VRF Sub ID.
- **`FundSubscription`**: Funds the subscription with LINK (uses `transferAndCall` on Sepolia, or direct funding on Mocks).
- **`AddConsumer`**: Whitelists the Raffle contract to use the subscription.

### [`script/DeployRaffle.s.sol`](./script/DeployRaffle.s.sol)

Main deployment script that orchestrates the entire deployment process, including creating subscriptions and adding consumers.

### [`test/unit/RaffleTest.t.sol`](./test/unit/RaffleTest.t.sol)

Comprehensive unit tests covering all contract functionality, edge cases, and security considerations.

### [`test/mocks/LinkToken.sol`](./test/mocks/LinkToken.sol)

Mock LINK token contract for local testing without requiring actual LINK tokens.

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.

## 🤝 Acknowledgments

- **Patrick Collins & Cyfrin Updraft:** For the incredible educational resources.
- **Chainlink Labs:** For the VRF and Automation infrastructure.
- **Foundry Team:** For the blazing fast development framework.

---

**Made with ❤️ using Foundry and Chainlink**