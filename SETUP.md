# VitalFi Setup Guide

**Complete guide to setting up the VitalFi development environment and deploying to Solana networks.**

Built by Credit Markets for transparent, institutional-grade medical receivables financing.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Initial Setup](#initial-setup)
3. [Local Development](#local-development)
4. [Devnet Deployment](#devnet-deployment)
5. [Testing](#testing)
6. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Tools

#### 1. Node.js & Package Managers
```bash
# Install Node.js 22+ (using nvm recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 22
nvm use 22

# Verify installation
node --version  # Should be v22.x.x
npm --version   # Should be 10.x.x

# Install Yarn
npm install -g yarn
```

#### 2. Rust & Cargo
```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Add to PATH (if not done automatically)
source $HOME/.cargo/env

# Verify installation
rustc --version  # Should be 1.79+
cargo --version
```

#### 3. Solana CLI
```bash
# Install Solana CLI
sh -c "$(curl -sSfL https://release.solana.com/v1.18.0/install)"

# Add to PATH
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"

# Verify installation
solana --version  # Should be 1.18+

# Configure CLI for devnet
solana config set --url https://api.devnet.solana.com

# Create a keypair (if you don't have one)
solana-keygen new --outfile ~/.config/solana/id.json

# Check your address
solana address

# Airdrop some SOL for testing
solana airdrop 2
```

#### 4. Anchor CLI
```bash
# Install Anchor CLI
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force
avm install 0.31.1
avm use 0.31.1

# Verify installation
anchor --version  # Should be 0.31.1
```

#### 5. Git
```bash
# Install Git (if not already installed)
# macOS
brew install git

# Linux (Ubuntu/Debian)
sudo apt-get install git

# Verify installation
git --version
```

### Optional Tools

#### Redis (for local backend testing)
```bash
# macOS
brew install redis
brew services start redis

# Linux
sudo apt-get install redis-server
sudo systemctl start redis

# Verify
redis-cli ping  # Should return "PONG"
```

#### Vercel CLI (for backend deployment)
```bash
npm install -g vercel

# Login to Vercel
vercel login
```

---

## Initial Setup

### 1. Clone the Repository

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/credit-markets/cypherpunk.git
cd cypherpunk

# Or if already cloned without submodules
git submodule update --init --recursive
```

### 2. Verify Submodules

```bash
# Check submodule status
git submodule status

# Should show:
# [commit-hash] programs (heads/dev)
# [commit-hash] backend (heads/dev)
# [commit-hash] app (heads/dev)
```

---

## Local Development

### Part 1: Smart Contracts (Programs)

#### Install Dependencies
```bash
cd programs
yarn install
```

#### Build Programs
```bash
# Clean previous builds
anchor clean

# Build (this will take a few minutes on first run)
anchor build

# Verify build artifacts
ls -la target/deploy/
# Should contain: vitalfi_vault.so, vitalfi_vault-keypair.json

# Check program ID
anchor keys list
# Should show: vitalfi_vault: 146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj
```

#### Configure Program ID
```bash
# If you need to generate a new program ID (for custom deployment)
solana-keygen new -o target/deploy/vitalfi_vault-keypair.json

# Update Anchor.toml with new address
anchor keys sync

# Rebuild to update program ID in code
anchor build
```

#### Run Tests
```bash
# Start local validator (in a separate terminal)
solana-test-validator

# Run all tests
anchor test --skip-local-validator

# Run specific test file
anchor test --skip-local-validator -- tests/happy-path.spec.ts

# Run with detailed logs
ANCHOR_LOG=true anchor test --skip-local-validator
```

#### Deploy to Localnet
```bash
# Make sure local validator is running
solana-test-validator

# Deploy program
anchor deploy --provider.cluster localnet

# Verify deployment
solana program show 146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj
```

---

### Part 2: Backend (Indexing Layer)

#### Install Dependencies
```bash
cd ../backend
yarn install
```

#### Configure Environment
```bash
# Copy example env file
cp .env.example .env.local

# Edit .env.local with your values
nano .env.local
```

**Required Environment Variables:**
```env
# Redis Connection (use local Redis or Vercel KV)
REDIS_URL=redis://localhost:6379

# Helius Configuration
HELIUS_WEBHOOK_SECRET=your-secret-token-here
HELIUS_API_KEY=your-helius-api-key

# VitalFi Program Configuration
VITALFI_PROGRAM_ID=146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj
NEXT_PUBLIC_SOLANA_RPC_ENDPOINT=http://localhost:8899

# Cache Configuration
CACHE_TTL=30
STORAGE_PREFIX=vitalfi:
ACTIVITY_TTL_DAYS=30
```

#### Run Backend Locally
```bash
# Start development server
yarn dev

# Backend will be available at http://localhost:3000

# Test health endpoint
curl http://localhost:3000/api/health
# Should return: {"ok":true,"kv":true,"timestamp":"..."}
```

#### Test API Endpoints
```bash
# Get vaults by authority
curl "http://localhost:3000/api/vaults?authority=YOUR_PUBKEY"

# Get positions by owner
curl "http://localhost:3000/api/positions?owner=YOUR_PUBKEY"

# Get activity feed
curl "http://localhost:3000/api/activity?vault=VAULT_PDA"
```

#### Setup Helius Webhook (Optional for local testing)
```bash
# Install ngrok for local webhook testing
brew install ngrok  # macOS
# or download from https://ngrok.com

# Start ngrok tunnel
ngrok http 3000

# Use the ngrok URL to configure Helius webhook
# Example: https://abc123.ngrok.io/api/webhooks/helius?token=your-secret
```

---

### Part 3: Frontend (App)

#### Install Dependencies
```bash
cd ../app
npm install
```

#### Configure Environment
```bash
# Copy example env file
cp .env.example .env.local

# Edit .env.local
nano .env.local
```

**Required Environment Variables:**
```env
# Program Configuration
NEXT_PUBLIC_PROGRAM_ID=146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj
NEXT_PUBLIC_VAULT_AUTHORITY=YOUR_AUTHORITY_PUBKEY_HERE

# Network Configuration
NEXT_PUBLIC_SOLANA_NETWORK=localnet
NEXT_PUBLIC_SOLANA_RPC_ENDPOINT=http://localhost:8899

# Backend API
NEXT_PUBLIC_API_URL=http://localhost:3000
```

#### Run Frontend Locally
```bash
# Start development server
npm run dev

# App will be available at http://localhost:3001

# Build for production (optional)
npm run build
npm start
```

#### Connect Wallet
1. Open http://localhost:3001 in your browser
2. Click "Connect Wallet" button
3. Select Phantom or Solflare
4. Approve connection

**For local testing, configure your wallet:**
- Network: Localhost
- RPC URL: http://localhost:8899

---

## Devnet Deployment

### Step 1: Prepare Devnet Wallet

```bash
# Switch to devnet
solana config set --url https://api.devnet.solana.com

# Check your balance
solana balance

# Airdrop SOL if needed (max 2 SOL per request)
solana airdrop 2

# For larger amounts, use the devnet faucet
# https://faucet.solana.com
```

### Step 2: Deploy Programs to Devnet

```bash
cd programs

# Build for devnet
anchor build

# Deploy (this will cost ~2-5 SOL for rent)
anchor deploy --provider.cluster devnet

# Verify deployment
solana program show 146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj --url devnet

# Run tests against devnet (optional)
anchor test --provider.cluster devnet --skip-deploy
```

### Step 3: Deploy Backend to Vercel

```bash
cd ../backend

# Create Vercel project
vercel

# Follow prompts:
# - Link to existing project or create new
# - Set root directory to "backend"

# Add environment variables to Vercel
vercel env add HELIUS_WEBHOOK_SECRET
vercel env add HELIUS_API_KEY
vercel env add VITALFI_PROGRAM_ID

# Add Vercel KV (Redis)
# 1. Go to Vercel dashboard
# 2. Storage tab
# 3. Create KV store
# 4. Link to your project (auto-adds REDIS_URL)

# Deploy to production
vercel --prod

# Note your deployment URL: https://your-project.vercel.app
```

### Step 4: Configure Helius Webhook

```bash
# Go to Helius Dashboard (https://dev.helius.xyz)
# 1. Create new webhook
# 2. Webhook URL: https://your-project.vercel.app/api/webhooks/helius?token=YOUR_SECRET
# 3. Webhook Type: Enhanced Transaction
# 4. Account Addresses: Add your vault PDAs or program ID
# 5. Transaction Types: All
# 6. Webhook Secret: Same as HELIUS_WEBHOOK_SECRET

# Test webhook
curl -X POST "https://your-project.vercel.app/api/webhooks/helius?token=YOUR_SECRET" \
  -H "Content-Type: application/json" \
  -d '[]'
```

### Step 5: Deploy Frontend to Vercel

```bash
cd ../app

# Update environment variables for devnet
# Edit .env.production
cat > .env.production << EOF
NEXT_PUBLIC_PROGRAM_ID=146hbPFqGb9a3v3t1BtkmftNeSNqXzoydzVPk95YtJNj
NEXT_PUBLIC_VAULT_AUTHORITY=YOUR_AUTHORITY_PUBKEY
NEXT_PUBLIC_SOLANA_NETWORK=devnet
NEXT_PUBLIC_SOLANA_RPC_ENDPOINT=https://api.devnet.solana.com
NEXT_PUBLIC_API_URL=https://your-backend.vercel.app
EOF

# Deploy to Vercel
vercel

# Set environment variables in Vercel dashboard
vercel env add NEXT_PUBLIC_PROGRAM_ID production
vercel env add NEXT_PUBLIC_VAULT_AUTHORITY production
vercel env add NEXT_PUBLIC_SOLANA_NETWORK production
vercel env add NEXT_PUBLIC_API_URL production

# Deploy to production
vercel --prod

# Your app is now live at: https://your-app.vercel.app
```

---

## Testing

### Unit Tests (Smart Contracts)

```bash
cd programs

# Run all tests
anchor test

# Run specific test suite
anchor test -- tests/happy-path.spec.ts
anchor test -- tests/security.spec.ts
anchor test -- tests/validation.spec.ts

# Run with coverage (requires additional setup)
# TODO: Add coverage tooling
```

### Integration Tests (Backend)

```bash
cd backend

# Run all tests
npm test

# Run specific test file
npm test -- src/lib/api.test.ts

# Run with watch mode
npm test -- --watch
```

### E2E Tests (Full Stack)

```bash
# 1. Start local validator
solana-test-validator

# 2. Deploy programs (in separate terminal)
cd programs
anchor deploy --provider.cluster localnet

# 3. Start backend (in separate terminal)
cd backend
yarn dev

# 4. Start frontend (in separate terminal)
cd app
npm run dev

# 5. Run E2E tests (TODO: Add Playwright/Cypress)
# npm run test:e2e
```

---

## Troubleshooting

### Common Issues

#### 1. "Anchor build failed"
```bash
# Solution: Clean and rebuild
anchor clean
rm -rf target/
anchor build
```

#### 2. "Program deploy failed - insufficient funds"
```bash
# Solution: Airdrop more SOL
solana airdrop 2

# Or for devnet, use faucet
# https://faucet.solana.com
```

#### 3. "Redis connection failed"
```bash
# Solution: Check if Redis is running
redis-cli ping

# If not running, start Redis
brew services start redis  # macOS
sudo systemctl start redis  # Linux
```

#### 4. "Wallet connection failed in browser"
```bash
# Solution:
# 1. Make sure wallet extension is installed
# 2. Check wallet is set to correct network (localnet/devnet)
# 3. Clear browser cache
# 4. Check console for CORS errors
```

#### 5. "Transaction failed - blockhash not found"
```bash
# Solution: This happens on devnet sometimes
# 1. Wait a few seconds and retry
# 2. Increase transaction confirmation timeout
# 3. Use a different RPC endpoint
```

#### 6. "Program account not found"
```bash
# Solution: Make sure program is deployed
solana program show YOUR_PROGRAM_ID

# Redeploy if needed
anchor deploy
```

#### 7. "Backend API returns 500"
```bash
# Solution: Check backend logs
vercel logs  # For production

# Or check local console output
# Look for Redis connection errors, missing env vars, etc.
```

### Debug Tips

#### Enable Verbose Logging
```bash
# Anchor tests
ANCHOR_LOG=true anchor test

# Solana CLI
solana -v program deploy target/deploy/vitalfi_vault.so

# Backend (local)
DEBUG=* yarn dev
```

#### Check Program Logs
```bash
# Follow program logs on localnet
solana logs | grep "Program 146hb"

# Check specific transaction
solana confirm -v TRANSACTION_SIGNATURE
```

#### Inspect Accounts
```bash
# View vault account
solana account VAULT_PDA_ADDRESS

# View position account
solana account POSITION_PDA_ADDRESS

# Decode account data (use Anchor)
anchor account vitalfi_vault.Vault VAULT_PDA_ADDRESS
```

---

## Next Steps

After successful setup:
1. **Read the Architecture Doc:** [ARCHITECTURE.md](./ARCHITECTURE.md)
2. **Test the Flow:** Create a vault, deposit, finalize, mature, claim
3. **Explore the Code:** Check out each submodule's README
4. **Customize:** Modify parameters, add features, experiment
5. **Deploy to Mainnet:** Follow mainnet deployment guide (TODO)

---

## Additional Resources

- **VitalFi Docs:** https://docs.vitalfi.lat
- **Landing Page:** https://vitalfi.lat
- **Solana Docs:** https://docs.solana.com
- **Anchor Book:** https://book.anchor-lang.com
- **Helius Docs:** https://docs.helius.xyz
- **Vercel Docs:** https://vercel.com/docs

---

**Need Help?**

- **GitHub Issues:** Open an issue in the repo
- **Email:** team@vitalfi.io
- **Website:** https://vitalfi.lat

---

**Powered by Credit Markets | Built on Solana**
