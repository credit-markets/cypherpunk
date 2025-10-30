# VitalFi: Earn Yield. Empower Healthcare.

**Solana Cypherpunk Hackathon Submission**

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
- Programs-enforced payouts at maturity (~75 days)
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
│                         User/Investor                       │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      VitalFi App (Frontend)                 │
│   Next.js 15 + React 19 + Solana Wallet Adapter             │
│   • Vault Dashboard & Deposits                              │
│   • Portfolio Tracking                                      │
│   • Transparency Pages                                      │
└─────────────┬────────────────────────────┬──────────────────┘
              │                            │
              │ Read (APIs)                │ Write (Transactions)
              ▼                            ▼
┌─────────────────────────┐  ┌───────────────────────────────┐
│  VitalFi Backend        │  │    Solana Blockchain          │
│  (Indexing Layer)       │  │    VitalFi Programs           │
│                         │  │                               │
│  • Vercel Functions     │  │  • Anchor Programs            │
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

### 1. [programs/](https://github.com/credit-markets/vitalfi-programs/tree/cypherpunk) - Solana Programs

### 2. [backend/](https://github.com/credit-markets/vitalfi-backend/tree/cypherpunk) - Indexing & Caching Layer

### 3. [app/](https://github.com/credit-markets/vitalfi-app/tree/cypherpunk) - Frontend DApp

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

### Setup Programs

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

### Programs Tests

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
- Anchor Framework (Rust)
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

## License

MIT License - See individual repositories for details.

---

**Powered by Credit Markets | Built on Solana**

_Earn Yield. Empower Healthcare._
