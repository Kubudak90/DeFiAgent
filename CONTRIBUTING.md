# Contributing to DeFiAgent

Thank you for your interest in contributing to DeFiAgent! This document provides guidelines and instructions for contributing to this autonomous AI-operated DeFi vault on Base.

## Table of Contents

- [Development Setup](#development-setup)
- [Project Architecture](#project-architecture)
- [Coding Standards](#coding-standards)
- [Testing Requirements](#testing-requirements)
- [Security Considerations](#security-considerations)
- [Areas for Contribution](#areas-for-contribution)
- [Submitting Changes](#submitting-changes)
- [Security Disclosure](#security-disclosure)

## Development Setup

### Prerequisites

- [Node.js](https://nodejs.org/) v18+ and [pnpm](https://pnpm.io/)
- [Foundry](https://book.getfoundry.sh/getting-started/installation) for smart contract development
- Git

### Installation

1. Clone the repository:
```bash
git clone https://github.com/cypherpulse/DeFiAgent.git
cd DeFiAgent
```

2. Install frontend dependencies:
```bash
cd frontend
pnpm install
cd ..
```

3. Set up Foundry for smart contracts:
```bash
cd solidityContract
forge install
cd ..
```

4. Create environment files:
```bash
# In frontend/
cp .env.example .env.local
# Edit .env.local with your configuration
```

## Project Architecture

DeFiAgent combines AI agents with DeFi vaults on Base:

### Smart Contract Architecture (`solidityContract/`)

**Core Contract: `DeFiAgent.sol`**
- **Ownership**: Ownable pattern for admin functions
- **Reentrancy Protection**: All state-changing functions use ReentrancyGuard
- **Agent System**: Owner can grant/revoke AI agent permissions
- **Yield Distribution**: 0.75% performance fee on harvested yield
- **Multi-Token Support**: ETH and ERC20 deposits

**Key Components:**
```
DeFiAgent/
├── src/
│   └── DeFiAgent.sol          # Main vault contract
├── test/
│   └── DeFiAgent.t.sol        # Comprehensive test suite
└── script/
    └── DeployDeFiAgent.s.sol  # Deployment scripts
```

**Contract Features:**
- `depositETH()`: Receive ETH deposits from users
- `depositERC20(token, amount)`: Receive ERC20 token deposits
- `grantAgent(agent)`: Authorize AI agent to harvest yield
- `revokeAgent(agent)`: Remove agent authorization
- `harvestYield()`: AI agent harvests and compounds yield (collects 0.75% fee)
- `withdrawETH(amount)`: Users withdraw their ETH deposits
- `withdrawERC20(token, amount)`: Users withdraw ERC20 deposits

### Frontend Architecture (`frontend/`)

- **Framework**: React with TypeScript
- **Styling**: Tailwind CSS
- **Web3**: wagmi/viem for blockchain interactions
- **Deployment**: Vercel

## Coding Standards

### Solidity

- **Version**: `^0.8.24`
- **Style Guide**: Follow [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html)
- **Imports**: Group OpenZeppelin imports first, then local imports
- **Naming**:
  - Contracts: `PascalCase`
  - Functions/Variables: `camelCase`
  - Constants: `UPPER_SNAKE_CASE`
  - Private variables: `_leadingUnderscore`

**Example:**
```solidity
// Good
uint256 public constant PERFORMANCE_FEE_BPS = 75;
mapping(address => uint256) private _userDeposits;

function harvestYield() external onlyOwnerOrAgent nonReentrant {
    // Implementation
}
```

### TypeScript/React

- Use TypeScript for all new code
- Follow existing component patterns
- Use functional components with hooks
- Properly type all props and state

## Testing Requirements

### Smart Contract Tests

**Minimum Coverage**: 80% line coverage required

**Test Organization** (`solidityContract/test/`):
```solidity
// DeFiAgent.t.sol structure
contract DeFiAgentTest is Test {
    // Setup
    function setUp() public {
        // Deploy contracts
    }
    
    // Deposit tests
    function test_DepositETH() public { }
    function test_DepositERC20() public { }
    
    // Agent tests
    function test_GrantAgent() public { }
    function test_RevokeAgent() public { }
    
    // Yield harvest tests
    function test_HarvestYield() public { }
    function test_HarvestYieldFeeCalculation() public { }
    
    // Withdrawal tests
    function test_WithdrawETH() public { }
    function test_WithdrawERC20() public { }
    
    // Security tests
    function test_ReentrancyProtection() public { }
    function test_OnlyOwnerOrAgent() public { }
}
```

**Running Tests:**
```bash
cd solidityContract
forge test
forge test --coverage  # Check coverage
```

### Frontend Tests

```bash
cd frontend
pnpm test
```

## Security Considerations

### Critical Invariants

1. **User deposits must always be withdrawable**: `userDepositETH[user] + userDepositERC20[user][token] <= contract balance`
2. **Performance fee must never exceed 0.75%**: `PERFORMANCE_FEE_BPS = 75` (basis points)
3. **Only authorized agents can harvest**: `isAgent[agent] == true` required for yield harvesting
4. **Reentrancy protection**: All external calls follow checks-effects-interactions pattern

### Common Vulnerabilities to Avoid

- **Reentrancy**: Always use `nonReentrant` modifier for functions with external calls
- **Access Control**: Verify `onlyOwnerOrAgent` modifier on sensitive functions
- **Integer Overflow**: Use Solidity 0.8+ built-in overflow protection
- **Front-running**: Consider MEV protection for yield harvesting

### Agent System Security

- Agents have significant power (can harvest yield)
- Agent addresses should be verified and trusted
- Consider multi-sig or timelock for agent management in production
- Monitor agent activity for anomalies

## Areas for Contribution

### High Priority

1. **Yield Strategy Integration**
   - Integrate with Aerodrome liquidity pools
   - Add Moonwell lending strategies
   - Implement Aave V3 yield farming
   - Support Beefy Finance auto-compounding vaults

2. **Price Oracle Integration**
   - Add Chainlink price feeds for accurate deposit valuation
   - Support multiple ERC20 tokens with proper pricing
   - Implement deposit value tracking in USD terms

3. **Gas Optimization**
   - Optimize storage layout
   - Batch operations where possible
   - Reduce redundant state reads

### Medium Priority

4. **Frontend Improvements**
   - Add deposit/withdraw history UI
   - Implement yield analytics dashboard
   - Add agent activity monitoring
   - Mobile-responsive design improvements

5. **Testing Expansion**
   - Fuzz testing for edge cases
   - Integration tests with real protocols
   - Frontend E2E tests

6. **Documentation**
   - API documentation for contract functions
   - Agent integration guide
   - Deployment guides for different networks

### Low Priority

7. **Additional Features**
   - Multi-sig agent authorization
   - Timelock for critical operations
   - Emergency pause functionality
   - Deposit/withdrawal limits

## Submitting Changes

1. **Fork and Branch**: Create a feature branch from `main`
2. **Follow Standards**: Adhere to coding standards above
3. **Add Tests**: Include tests for all new functionality
4. **Update Documentation**: Update README/docs if needed
5. **Commit Messages**: Use clear, descriptive commit messages
   - `feat: add Aerodrome yield strategy`
   - `fix: correct fee calculation in harvestYield`
   - `docs: update agent integration guide`
6. **Pull Request**: Open PR with detailed description

## Security Disclosure

**Please do not file public issues for security vulnerabilities.**

Instead, report security issues privately:
- Email: [security contact]
- Include detailed description and reproduction steps
- Allow time for response before public disclosure

We follow responsible disclosure practices and will acknowledge reporters (with permission).

---

**Questions?** Open a [GitHub Discussion](https://github.com/cypherpulse/DeFiAgent/discussions) or reach out to the maintainers.

Thank you for helping make DeFiAgent better! 🚀