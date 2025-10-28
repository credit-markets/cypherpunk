# VitalFi: Earn Yield. Empower Healthcare.

**Solana Cypherpunk Hackathon Submission | Built by Credit Markets**

VitalFi connects liquidity providers directly to Brazilian medical receivables, eliminating costly intermediaries while delivering institutional-grade yields. Built on Solana for speed, transparency, and global accessibility.

## The Problem

Brazilian healthcare providers face a critical liquidity gap:

- **Payment Delays:** Medical receivables from insurers and hospitals take **~75 days** to settle
- **Expensive Capital:** Traditional factoring charges **15-25% annually** in fees
- **Opacity:** Limited visibility into collateral quality and risk assessment
- **Restricted Access:** Fragmented capital markets limit access to competitive financing

**Market Size:** $627 billion Brazilian healthcare market with 500,000+ healthcare providers

## The Solution

VitalFi delivers **up to 16% APY** backed by institutional-grade medical receivables:

**For Liquidity Providers:**

- Earn predictable yields from real-world healthcare financing
- Full on-chain transparency of collateral and payer creditworthiness
- Smart contract-enforced payouts at maturity (~75 days)
- Deposit USDT, earn yield—no active management required

**For Healthcare Providers:**

- Receive payment within **24 hours** instead of 75 days
- Eliminate costly bridge financing and factoring fees
- Access global DeFi liquidity pools
- Transparent, automated settlement

**Built on Solana:** Sub-second finality, sub-cent fees, and global 24/7 access

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

### For Liquidity Providers

- **Institutional-Grade Yields:** Up to 16% APY backed by verified medical receivables
- **Real-World Asset Exposure:** Diversify beyond crypto-native yields into healthcare financing
- **Full Transparency:** On-chain verification of all receivables, payer creditworthiness, and currency hedges
- **Predictable Maturity:** ~75-day lock periods with defined payout schedules
- **Risk Mitigation:** Whitelisted institutional payers with 98%+ historical recovery rates
- **Simple UX:** Deposit USDT → Lock → Earn at maturity

### For Healthcare Providers

- **Speed:** Get paid in **24 hours** instead of waiting 75+ days for insurance settlements
- **Lower Costs:** Eliminate 15-25% annual factoring fees
- **Capital Efficiency:** Improve cash flow for operational needs (payroll, supplies, equipment)
- **Global Liquidity:** Access DeFi capital pools without geographic constraints

### Technical Excellence

- **Non-Custodial Security:** PDA-based smart contracts ensure users maintain full custody
- **Threshold Funding:** 2/3 capacity requirement protects both LPs and healthcare providers
- **Automatic Refunds:** Failed vaults return 100% of deposits via smart contract
- **Currency Risk Management:** Transparent BRL/USD hedging via NDFs and forwards
- **Real-Time Indexing:** Instant transaction visibility via Helius webhooks
- **Production-Grade:** Comprehensive test coverage and error handling

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
git clone --recurse-submodules https://github.com/credit-markets/cypherpunk.git
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

- **Frontend:** [\[Demo URL\]](https://app.vitalfi.lat/)
- **Program ID:** `146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj`
- **Network:** Solana Devnet
- **Backend API:** [\[API URL\]](https://api.vitalfi.lat/)

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

## Why Solana?

VitalFi leverages Solana's unique capabilities for real-world asset financing:

- **Speed:** Sub-second finality enables same-day medical provider payments
- **Cost:** Sub-cent transaction fees make small deposits economically viable
- **Throughput:** Supports thousands of concurrent vaults and transactions
- **Transparency:** All vault parameters and collateral verifiable on-chain
- **Global Access:** Borderless capital for Brazilian healthcare, 24/7

## About Credit Markets

Credit Markets specializes in structured finance and real-world asset tokenization. We originate and service medical receivables in Brazil, maintaining partnerships with institutional payers and healthcare providers. VitalFi is our flagship DeFi product, connecting global liquidity to emerging market healthcare financing.

**Key Metrics:**

- 98%+ historical receivable recovery rate
- 12+ whitelisted institutional payers (average AA+ rating)
- $627B total addressable market
- 500K+ potential healthcare provider users

## License

MIT License - See individual repositories for details.

---

**Powered by Credit Markets | Built on Solana**

_Earn Yield. Empower Healthcare._
