# VitalFi: Decentralized Medical Receivables Financing on Solana

**Hackathon Submission - Solana Cypherpunk Track**

VitalFi is a blockchain-based crowdfunding platform that connects DeFi investors with healthcare providers in Brazil, enabling transparent and efficient financing of medical receivables through Solana smart contracts.

## Problem Statement

Healthcare providers in Brazil face significant cash flow challenges:
- Medical receivables from insurance companies can take 60-90+ days to settle
- Traditional factoring services charge high fees (15-25% annually)
- Lack of transparency in collateral and risk assessment
- Limited access to global capital markets

## Solution

VitalFi creates a decentralized marketplace where:
- Healthcare providers can securitize their receivables into transparent vaults
- Global investors earn predictable yields by providing liquidity
- Smart contracts ensure trustless fund management and automatic payouts
- Full transparency into collateral, hedge positions, and receivables
- Lower costs through blockchain efficiency and direct investor access

## Architecture

VitalFi consists of three integrated components:

```
┌─────────────────────────────────────────────────────────────┐
│                         User/Investor                        │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      VitalFi App (Frontend)                  │
│   Next.js 15 + React 19 + Solana Wallet Adapter            │
│   • Vault Dashboard & Deposits                              │
│   • Portfolio Tracking                                       │
│   • Transparency Pages                                       │
└─────────────┬────────────────────────────┬──────────────────┘
              │                            │
              │ Read (APIs)                │ Write (Transactions)
              ▼                            ▼
┌─────────────────────────┐  ┌───────────────────────────────┐
│  VitalFi Backend        │  │    Solana Blockchain          │
│  (Indexing Layer)       │  │    VitalFi Programs           │
│                         │  │                               │
│  • Vercel Functions     │  │  • Anchor Smart Contracts     │
│  • Redis Cache          │  │  • Vault Management           │
│  • REST APIs            │  │  • Position Tracking          │
│  • Activity Feed        │  │  • Trustless Settlements      │
└─────────────────────────┘  └───────────────────────────────┘
              ▲                            │
              │                            │
              └────────────────────────────┘
                   Helius Webhooks
                   (Event Indexing)
```

## Repository Structure

This repository contains three submodules:

### 1. [programs/](./programs) - Solana Smart Contracts
- **Language:** Rust with Anchor Framework 0.31.1
- **Program ID:** `146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj`
- **Features:**
  - Threshold-based crowdfunding (requires 2/3 capacity)
  - SPL token deposits with minimum amount enforcement
  - Deterministic PDA-based account architecture
  - Automatic refunds for failed funding rounds
  - Proportional payout distribution at maturity
  - Comprehensive event emissions for indexing

### 2. [backend/](./backend) - Indexing & Caching Layer
- **Stack:** TypeScript + Vercel Serverless + Redis
- **Purpose:**
  - Real-time blockchain event processing via Helius webhooks
  - Efficient caching with ETag support
  - Paginated REST APIs for vaults, positions, and activity
  - Optimized query performance with Redis sorted sets

### 3. [app/](./app) - Frontend DApp
- **Stack:** Next.js 15 + React 19 + TanStack Query + Tailwind CSS
- **Features:**
  - Multi-wallet support (Phantom, Solflare)
  - Real-time portfolio tracking
  - Vault transparency (collateral, hedges, receivables)
  - Responsive design with dark mode
  - Smart polling for transaction confirmations

## Key Features

### For Investors
- **Fixed Yields:** Predictable returns backed by medical receivables
- **Transparent Collateral:** Full visibility into underlying assets
- **Automatic Payouts:** Smart contract-based claim process
- **No Lock-in Risk:** Funds released at maturity with guaranteed access
- **Global Access:** Anyone with a Solana wallet can participate

### For Healthcare Providers
- **Lower Costs:** Reduced financing fees compared to traditional factoring
- **Faster Access:** Immediate liquidity after vault funding succeeds
- **Flexible Terms:** Customizable vault parameters (cap, duration, APY)
- **Global Capital:** Access to worldwide DeFi liquidity

### Technical Highlights
- **Non-custodial:** Users never lose control of funds (PDA-based custody)
- **Threshold Funding:** 2/3 capacity requirement ensures sufficient liquidity
- **Refund Protection:** Automatic refunds if funding fails
- **Event-Driven:** Real-time indexing for instant UI updates
- **Production-Ready:** Comprehensive error handling and test coverage

## How It Works

### Vault Lifecycle

1. **Creation (Funding Phase)**
   - Authority initializes vault with parameters (cap, APY, maturity date, asset mint)
   - Vault opens for deposits with minimum amount enforcement
   - Users deposit SPL tokens during funding window

2. **Finalization**
   - After funding period ends, authority finalizes vault
   - **Success Path:** If deposits ≥ 2/3 capacity → funds transferred to authority
   - **Failure Path:** If deposits < 2/3 capacity → vault marked for refunds

3. **Active Phase (Off-Chain)**
   - Authority uses funds to finance medical receivables in Brazil
   - Currency hedging positions established (NDF, forwards, swaps)
   - Receivables mature and generate returns

4. **Maturity**
   - Authority returns principal + yield to vault
   - Vault marked as "Matured"
   - Users can claim proportional payouts

5. **Claiming**
   - Users call `claim()` to receive their share
   - Formula: `payout = deposited × (payout_num / payout_den)`
   - Supports multiple partial claims

6. **Closure**
   - After all claims processed, authority closes vault
   - Rent reclaimed, vault account freed

## Getting Started

### Prerequisites
- Node.js 22+
- Rust 1.79+
- Solana CLI 1.18+
- Anchor CLI 0.31+
- Yarn or npm
- Solana wallet (Phantom or Solflare)

### Clone Repository with Submodules

```bash
git clone --recurse-submodules https://github.com/YOUR_ORG/cypherpunk.git
cd cypherpunk

# Or if already cloned:
git submodule update --init --recursive
```

### Setup Programs (Smart Contracts)

```bash
cd programs

# Install dependencies
yarn install

# Build programs
anchor build

# Run tests
anchor test

# Deploy to devnet (requires funded wallet)
anchor deploy --provider.cluster devnet
```

### Setup Backend (Indexing Layer)

```bash
cd backend

# Install dependencies
yarn install

# Configure environment
cp .env.example .env.local
# Edit .env.local with your values:
# - REDIS_URL (Vercel KV or local Redis)
# - HELIUS_WEBHOOK_SECRET
# - VITALFI_PROGRAM_ID

# Run locally
yarn dev

# Deploy to Vercel
vercel deploy
```

### Setup App (Frontend)

```bash
cd app

# Install dependencies
npm install

# Configure environment
cp .env.example .env.local
# Edit .env.local with your values:
# - NEXT_PUBLIC_PROGRAM_ID
# - NEXT_PUBLIC_VAULT_AUTHORITY
# - NEXT_PUBLIC_API_URL
# - NEXT_PUBLIC_SOLANA_NETWORK

# Run locally
npm run dev

# Build for production
npm run build
npm start
```

### Access the App

Open [http://localhost:3000](http://localhost:3000) and connect your Solana wallet.

## Testing

### Smart Contract Tests
```bash
cd programs
anchor test
```

Test coverage includes:
- Happy path scenarios (full vault lifecycle)
- Authority permission validation
- Security constraints (reentrancy, overflow protection)
- Input validation (timestamps, amounts, capacities)
- Edge cases (dust amounts, partial claims)

### Backend Tests
```bash
cd backend
npm test
```

### End-to-End Testing
1. Start local Solana validator: `solana-test-validator`
2. Deploy programs: `anchor deploy --provider.cluster localnet`
3. Start backend: `cd backend && yarn dev`
4. Start frontend: `cd app && npm run dev`
5. Test full user flow in browser

## Live Demo

- **Frontend:** [Demo URL]
- **Program ID:** `146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj`
- **Network:** Solana Devnet
- **Backend API:** [API URL]

## Use Cases

### Primary: Medical Receivables in Brazil
- Healthcare clinics securitize invoices from insurance companies
- Typical receivables: R$100K-R$2M per vault
- Maturity: 60-120 days
- Target APY: 12-18% annually
- Currency hedging via NDF contracts (BRL/USD)

### Future Expansion
- Other emerging markets with healthcare financing gaps
- Invoice factoring for medical equipment suppliers
- Pharmaceutical distribution financing
- Multi-country vault pooling

## Security Considerations

### Smart Contract Security
- **Audited Code:** Comprehensive test coverage with edge cases
- **PDA-based Custody:** Non-custodial architecture prevents rug pulls
- **Overflow Protection:** Checked arithmetic operations
- **Access Control:** Authority-only operations properly gated
- **Reentrancy Safe:** Single-threaded execution model

### Operational Security
- **Webhook Verification:** HMAC-based authentication for Helius webhooks
- **Input Validation:** Zod schemas for all API inputs
- **Rate Limiting:** Vercel edge protection
- **Environment Isolation:** Separate configs for localnet/devnet/mainnet

### Transparency Guarantees
- **On-Chain Verification:** All vault parameters and deposits publicly auditable
- **Receivables Disclosure:** Full collateral documentation available
- **Hedge Position Tracking:** Currency hedging transparently reported
- **Real-Time Events:** All state changes emit events for indexing

## Team

[Add team member information here]

## Roadmap

### Phase 1 (Current - Hackathon)
- ✅ Core smart contracts deployed on devnet
- ✅ Backend indexing with Redis caching
- ✅ Frontend DApp with wallet integration
- ✅ Transparency pages with receivables data

### Phase 2 (Q1 2025)
- [ ] Security audit by reputable firm
- [ ] Mainnet deployment with initial vaults
- [ ] Integration with Brazilian healthcare providers
- [ ] Fiat on/off-ramp partnerships

### Phase 3 (Q2 2025)
- [ ] Multi-currency support (USDC, USDT, BRL stablecoin)
- [ ] Secondary market for position trading
- [ ] DAO governance for vault approval
- [ ] Mobile app (iOS/Android)

### Phase 4 (Q3-Q4 2025)
- [ ] Expansion to other Latin American markets
- [ ] Institutional investor dashboard
- [ ] Credit scoring system for healthcare providers
- [ ] Insurance products for vault risk mitigation

## Technologies Used

### Blockchain Layer
- Solana (high throughput, low fees)
- Anchor Framework (Rust smart contracts)
- SPL Token Program (token standard)

### Backend
- TypeScript (type-safe development)
- Vercel Serverless (edge compute)
- Redis (Vercel KV for caching)
- Helius (blockchain indexing)

### Frontend
- Next.js 15 (React framework)
- TanStack Query (state management)
- Solana Wallet Adapter (multi-wallet)
- Tailwind CSS (styling)
- Recharts (data visualization)

### DevOps
- GitHub Actions (CI/CD)
- Vercel (deployment)
- Anchor CLI (program deployment)

## Contributing

We welcome contributions! Please see individual repositories for contribution guidelines:
- [programs/CONTRIBUTING.md](./programs/CONTRIBUTING.md)
- [backend/CONTRIBUTING.md](./backend/CONTRIBUTING.md)
- [app/CONTRIBUTING.md](./app/CONTRIBUTING.md)

## License

This project is licensed under the MIT License - see individual repositories for details.

## Support

- **Documentation:** [Wiki](https://github.com/YOUR_ORG/cypherpunk/wiki)
- **Discord:** [Join our community]
- **Twitter:** [@VitalFi]
- **Email:** team@vitalfi.io

## Acknowledgments

- Solana Foundation for the blockchain infrastructure
- Anchor framework team for smart contract tooling
- Helius for reliable blockchain indexing
- Brazilian healthcare providers for partnership and feedback
- The Cypherpunk community for inspiration

---

**Built with ❤️ for the Solana Cypherpunk Hackathon**

*Empowering healthcare finance through blockchain technology*
