# SilentPoll

**Anonymous voting platform with cryptographic privacy guarantees**

SilentPoll enables organizations, DAOs, and communities to conduct polls and votes where individual choices remain encrypted until results are tallied. Built on Zama FHEVM (Fully Homomorphic Encryption Virtual Machine), the platform ensures that vote tallies can be computed over encrypted ballots without ever decrypting individual votes.

---

## The Challenge

Modern voting systems face an inherent tension between transparency and privacy:

- **Public blockchains** offer auditability but expose every vote
- **Private databases** provide secrecy but lack verifiable integrity
- **Hybrid approaches** often require trusted intermediaries

**SilentPoll solves this** by combining blockchain verifiability with homomorphic encryption, allowing anyone to verify results while keeping individual votes private.

---

## How It Works

### Step 1: Poll Creation
A poll creator defines:
- Question or proposal
- Voting options (binary, multiple choice, ranked)
- Eligibility criteria (token holders, NFT owners, whitelist)
- Voting window (start/end times)

### Step 2: Encrypted Voting
Each voter:
1. Connects their wallet (proving eligibility)
2. Selects their choice
3. Encrypts the vote using Zama FHEVM public key
4. Submits encrypted vote to smart contract
5. Receives cryptographic receipt (proof of submission)

**Key Point**: The contract stores only encrypted votes. No one—not even validators—can see individual choices.

### Step 3: Homomorphic Tallying
Once voting closes:
1. Smart contract aggregates encrypted votes using FHE operations
2. Computes encrypted totals for each option
3. Produces encrypted result vector

### Step 4: Result Revelation
Threshold key holders decrypt results:
- Multiple independent parties hold key fragments
- Requires quorum to decrypt (prevents single point of failure)
- Final results published on-chain with verification proofs

### Step 5: Verification
Anyone can verify:
- All eligible votes were counted
- Results match encrypted commitments
- No votes were altered or double-counted

---

## Architecture

```
┌──────────────┐
│   Voter UI   │
│  (React App) │
└──────┬───────┘
       │ 1. Connect wallet
       │ 2. Select option
       │ 3. Encrypt vote
       ▼
┌─────────────────────────────────────┐
│      Zama FHEVM Smart Contract      │
│  ┌───────────────────────────────┐  │
│  │ Receive encrypted votes       │  │
│  │ Store: euint64[] ballots      │  │
│  │ Validate eligibility          │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │ Tally Phase                   │  │
│  │ Homomorphic sum over ballots  │  │
│  │ Result: euint64[] totals      │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │ Reveal Phase                  │  │
│  │ Threshold decryption          │  │
│  │ Publish: uint256[] results    │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
       │ Public events & proofs
       ▼
┌──────────────┐
│   Explorer   │
│ (Verification)│
└──────────────┘
```

---

## Core Features

### Privacy Guarantees
- **Individual Vote Privacy**: Votes encrypted until aggregation completes
- **No Linkability**: Cannot trace votes back to voters (when using anonymous wallets)
- **Zero-Knowledge Eligibility**: Prove eligibility without revealing identity

### Verifiability
- **Cryptographic Proofs**: Every poll produces verifiable proof artifacts
- **Public Auditing**: Anyone can verify vote counts match encrypted inputs
- **Immutable History**: All polls recorded on-chain for audit trails

### Flexibility
- **Multiple Question Types**: Binary, multiple choice, ranked voting
- **Custom Eligibility**: Token thresholds, NFT ownership, allowlists
- **Configurable Parameters**: Voting windows, reveal delays, quorum requirements

### User Experience
- **Wallet Integration**: MetaMask, WalletConnect, Coinbase Wallet
- **Mobile Support**: Responsive design for mobile voting
- **Gas Optimization**: Batch operations to reduce costs

---

## Use Cases

### DAO Governance
**Challenge**: Members hesitant to vote publicly due to social pressure  
**Solution**: Encrypted votes ensure genuine preferences without fear of retaliation  
**Example**: Treasury allocation proposals, protocol parameter changes

### Employee Surveys
**Challenge**: Need honest feedback but maintain confidentiality  
**Solution**: Encrypted responses aggregated into anonymous statistics  
**Example**: Performance reviews, workplace satisfaction polls

### Academic Research
**Challenge**: Collect sensitive survey data with verifiable integrity  
**Solution**: Encrypted submissions with cryptographic proof of data integrity  
**Example**: Medical history studies, political opinion research

### Community Decision-Making
**Challenge**: Transparent process but private individual choices  
**Solution**: Publicly verifiable results with private vote data  
**Example**: Neighborhood associations, online communities

---

## Technical Deep Dive

### FHE Data Types

SilentPoll uses Zama FHEVM's encrypted integer types:

```solidity
// Encrypted vote (one per option)
euint64 encryptedVote;

// Encrypted tally (aggregated)
euint64 encryptedTotal;

// Encrypted boolean (eligibility check)
ebool isEligible;
```

### Homomorphic Operations

**Addition**: Accumulate votes without decryption
```solidity
encryptedTotal = TFHE.add(encryptedTotal, encryptedVote);
```

**Comparison**: Determine winners over encrypted data
```solidity
ebool isGreater = TFHE.gt(optionA, optionB);
```

**Conditional**: Apply logic based on encrypted conditions
```solidity
encryptedResult = TFHE.cmux(condition, valueA, valueB);
```

### Gas Costs

| Operation | Estimated Gas |
|-----------|--------------|
| Submit encrypted vote | ~80,000 |
| Tally votes (100 votes) | ~500,000 |
| Tally votes (1000 votes) | ~2,000,000 |
| Reveal results | ~150,000 |
| Verify proof | ~40,000 |

*Note: Costs vary by network and FHEVM implementation*

---

## Security Model

### Threat Analysis

| Threat | Risk Level | Mitigation |
|--------|-----------|------------|
| Vote buying | Medium | Time-locked encryption (reveal delay) |
| Key compromise | High | Threshold key management, rotation |
| Front-running | Low | Commit-reveal scheme |
| DoS attacks | Medium | Gas limits, rate limiting |
| Sybil attacks | Medium | Token/NFT-based eligibility |
| Result manipulation | Low | Cryptographic proofs, public verification |

### Privacy Properties

1. **Confidentiality**: Individual votes remain encrypted during processing
2. **Anonymity**: Votes cannot be linked to voters (with proper wallet hygiene)
3. **Unlinkability**: Multiple votes from same voter appear independent
4. **Forward Secrecy**: Compromised keys don't reveal past votes (with rotation)

### Auditability Properties

1. **Completeness**: All eligible votes included in tally
2. **Correctness**: Final results match encrypted votes
3. **Soundness**: Invalid votes rejected
4. **Non-repudiation**: Voters cannot deny submission (via receipts)

---

## Getting Started

### Prerequisites

- Node.js 18+ and npm/yarn/pnpm
- Hardhat or Foundry for contract development
- MetaMask or compatible Web3 wallet
- Access to FHEVM-enabled network (Sepolia testnet)

### Installation

```bash
# Clone repository
git clone https://github.com/yourusername/silentpoll.git
cd silentpoll

# Install dependencies
npm install

# Compile contracts
npx hardhat compile

# Run tests
npx hardhat test
```

### Deployment

```bash
# Deploy to Sepolia testnet
npx hardhat run scripts/deploy.js --network sepolia

# Or use Foundry
forge script scripts/Deploy.s.sol --rpc-url $SEPOLIA_RPC_URL --broadcast
```

### Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

Visit `http://localhost:3000` and connect your wallet.

---

## API Reference

### Smart Contract Interface

```solidity
interface ISilentPoll {
    // Create a new poll
    function createPoll(
        string memory question,
        string[] memory options,
        uint256 startTime,
        uint256 endTime,
        address[] memory eligibleVoters
    ) external returns (uint256 pollId);
    
    // Submit encrypted vote
    function vote(
        uint256 pollId,
        bytes calldata encryptedChoice
    ) external;
    
    // Close voting and tally
    function closePoll(uint256 pollId) external;
    
    // Reveal results (threshold)
    function revealResults(
        uint256 pollId,
        bytes calldata decryptionKey
    ) external;
    
    // Get public results
    function getResults(uint256 pollId)
        external
        view
        returns (uint256[] memory);
    
    // Verify poll integrity
    function verifyPoll(uint256 pollId)
        external
        view
        returns (bool valid);
}
```

### Frontend SDK

```typescript
import { SilentPollClient } from '@silentpoll/sdk';

const client = new SilentPollClient({
  rpcUrl: 'https://sepolia.infura.io/v3/YOUR_KEY',
  contractAddress: '0x...',
});

// Create poll
const pollId = await client.createPoll({
  question: 'Should we implement feature X?',
  options: ['Yes', 'No', 'Abstain'],
  duration: 7 * 24 * 60 * 60, // 7 days
});

// Vote
await client.vote(pollId, encryptedChoice);

// Get results
const results = await client.getResults(pollId);
```

---

## Roadmap

### Q1 2025: Core Platform
- ✅ Encrypted vote submission
- ✅ Homomorphic tallying
- ✅ Result revelation
- 🔄 Threshold key management

### Q2 2025: Enhanced Features
- 🔄 Ranked choice voting
- 🔄 Delegation mechanisms
- 🔄 Mobile native app
- 🔄 Advanced analytics

### Q3 2025: Enterprise Features
- 📋 Multi-tenant support
- 📋 Custom eligibility modules
- 📋 API for integrations
- 📋 White-label deployment

### Q4 2025: Ecosystem Expansion
- 📋 Governance token
- 📋 Community-driven features
- 📋 Cross-chain compatibility
- 📋 Educational resources

---

## Contributing

We welcome contributions! Areas where help is especially needed:

- **Cryptography**: FHE optimizations, key management improvements
- **Security**: Audits, formal verification, threat modeling
- **Frontend**: UI/UX improvements, mobile optimization
- **Documentation**: Tutorials, API docs, use case examples
- **Testing**: Test coverage expansion, fuzzing, integration tests

**How to contribute:**
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

**Code standards:**
- Follow Solidity style guide for contracts
- Use TypeScript strict mode for frontend
- Maintain 85%+ test coverage
- Document public APIs

---

## License

MIT License - see [LICENSE](LICENSE) file for details.

---

## Acknowledgments

SilentPoll is built on cutting-edge privacy technology:

- **[Zama FHEVM](https://www.zama.ai/fhevm)**: Enables fully homomorphic encryption on Ethereum-compatible chains
- **[Zama](https://www.zama.ai/)**: Pioneering FHE research and tooling
- **Ethereum Foundation**: Blockchain infrastructure and standards

Special thanks to the Zama team for making FHE accessible to developers and advancing the state of privacy-preserving computation.

---

## FAQ

**Q: How private are votes if I use the same wallet address?**  
A: While votes are encrypted, using the same wallet address across multiple polls creates linkability. For maximum privacy, use different addresses or anonymous wallets.

**Q: Can votes be changed after submission?**  
A: No. Once a vote is submitted and confirmed on-chain, it cannot be modified. This prevents manipulation and ensures vote integrity.

**Q: What happens if the reveal key is lost?**  
A: Keys are managed using threshold cryptography. As long as a quorum of key holders is available, results can be revealed. Automatic time-locked reveal is available as a backup.

**Q: How do you prevent vote buying?**  
A: The time delay between vote submission and result revelation makes vote buying impractical—buyers can't verify which option was chosen until after the vote is final.

**Q: Is SilentPoll compatible with existing DAO frameworks?**  
A: Yes! SilentPoll can integrate with Snapshot, Tally, and other DAO tools through its API, providing encrypted voting as an alternative to transparent votes.

**Q: What's the maximum number of votes a poll can handle?**  
A: Limited primarily by gas costs. Current implementation handles 1,000+ votes efficiently. For larger polls, batch processing optimizations can be applied.

---

## Contact & Links

- **Repository**: [GitHub](https://github.com/yourusername/silentpoll)
- **Documentation**: [Full Docs](https://docs.silentpoll.io)
- **Discord**: [Community](https://discord.gg/silentpoll)
- **Twitter**: [@SilentPoll](https://twitter.com/silentpoll)

---

**SilentPoll** - Democracy needs privacy. Privacy needs verification. We deliver both.

_Powered by Zama FHEVM_

