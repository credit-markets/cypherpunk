# VitalFi Architecture Documentation

## System Overview

VitalFi is a three-tier decentralized application for medical receivables financing on Solana.

## Component Architecture

### Layer 1: Smart Contracts (On-Chain)

**Location:** `programs/vitalfi-vault`

#### Account Structure

```
┌─────────────────────────────────────────────┐
│            Vault PDA (210 bytes)            │
│  Seeds: ["vault", authority, vault_id]      │
├─────────────────────────────────────────────┤
│  • authority: Pubkey                        │
│  • vault_id: u64                            │
│  • status: VaultStatus (enum)               │
│  • cap: u64                                 │
│  • total_deposited: u64                     │
│  • total_claimed: u64                       │
│  • funding_end_ts: i64                      │
│  • maturity_ts: i64                         │
│  • target_apy_bps: u16                      │
│  • min_deposit: u64                         │
│  • payout_num: u128                         │
│  • payout_den: u128                         │
│  • asset_mint: Pubkey                       │
│  • vault_token_account: Pubkey              │
│  • created_at: i64                          │
│  • updated_at: i64                          │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│         Position PDA (89 bytes)             │
│  Seeds: ["position", vault, user]           │
├─────────────────────────────────────────────┤
│  • vault: Pubkey                            │
│  • owner: Pubkey                            │
│  • deposited: u64                           │
│  • claimed: u64                             │
│  • created_at: i64                          │
│  • updated_at: i64                          │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│  Vault Token Account PDA (SPL Token)        │
│  Seeds: ["vault_token", vault]              │
├─────────────────────────────────────────────┤
│  • Standard SPL Token Account               │
│  • Owned by Vault PDA                       │
│  • Mint: Configured asset_mint              │
└─────────────────────────────────────────────┘
```

#### State Machine

```
┌──────────┐
│ Funding  │ (status = 0)
└────┬─────┘
     │
     │ finalize_funding()
     │ (time > funding_end_ts)
     │
     ├─ deposited >= 2/3 cap ──┐
     │                          │
     ▼                          ▼
┌──────────┐              ┌──────────┐
│ Canceled │ (status = 2) │  Active  │ (status = 1)
└────┬─────┘              └────┬─────┘
     │                          │
     │ claim()                  │ mature_vault()
     │ (refunds)                │ (time > maturity_ts)
     │                          │
     │                          ▼
     │                    ┌──────────┐
     │                    │ Matured  │ (status = 3)
     │                    └────┬─────┘
     │                         │
     │                         │ claim()
     │                         │ (payouts)
     │                         │
     └─────────────────────────┤
                               │
                               │ close_vault()
                               │ (balance <= 1000)
                               │
                               ▼
                         ┌──────────┐
                         │  Closed  │ (status = 4)
                         └──────────┘
```

#### Instruction Flow

**1. initialize_vault()**

```
Input:
  • authority (signer)
  • vault (PDA, init)
  • vault_token_account (PDA, init)
  • asset_mint
  • params: {cap, funding_end_ts, maturity_ts, target_apy_bps, min_deposit, vault_id}

Validation:
  ✓ cap > 0 && cap <= MAX_VAULT_CAP
  ✓ funding_end_ts < maturity_ts
  ✓ min_deposit <= cap
  ✓ asset_mint is valid SPL token

Effects:
  → Create vault PDA
  → Create vault token account (owned by vault PDA)
  → Set status = Funding
  → Emit VaultCreated event
```

**2. deposit()**

```
Input:
  • user (signer)
  • vault (mut)
  • position (PDA, init-if-needed)
  • user_token_account
  • vault_token_account (mut)
  • amount

Validation:
  ✓ vault.status == Funding
  ✓ current_time < vault.funding_end_ts
  ✓ amount >= vault.min_deposit
  ✓ vault.total_deposited + amount <= vault.cap

Effects:
  → Transfer amount from user to vault (CPI)
  → Update/create position.deposited += amount
  → Update vault.total_deposited += amount
  → Emit DepositEvent
```

**3. finalize_funding()**

```
Input:
  • authority (signer)
  • vault (mut)
  • vault_token_account (mut)
  • authority_token_account (mut)

Validation:
  ✓ vault.status == Funding
  ✓ current_time >= vault.funding_end_ts OR vault.total_deposited >= vault.cap
  ✓ signer == vault.authority

Logic:
  if vault.total_deposited >= ceil(2 * vault.cap / 3):
    → Transfer all funds to authority (CPI)
    → Set status = Active
  else:
    → Set status = Canceled

Effects:
  → Emit FundingFinalized event
  → Emit AuthorityWithdraw event (if successful)
```

**4. mature_vault()**

```
Input:
  • authority (signer)
  • vault (mut)
  • vault_token_account (mut)
  • authority_token_account (mut)
  • payout_amount

Validation:
  ✓ vault.status == Active
  ✓ current_time >= vault.maturity_ts
  ✓ signer == vault.authority

Effects:
  → Transfer payout_amount from authority to vault (CPI)
  → Calculate: payout_num / payout_den = payout_amount / total_deposited
  → Set status = Matured
  → Emit Matured event
```

**5. claim()**

```
Input:
  • user (signer)
  • vault (mut)
  • position (mut)
  • user_token_account (mut)
  • vault_token_account (mut)

Validation:
  ✓ vault.status == Canceled OR Matured
  ✓ position.owner == user
  ✓ position.deposited > position.claimed

Logic:
  if vault.status == Canceled:
    entitled = position.deposited - position.claimed
  else: // Matured
    entitled = floor(position.deposited * vault.payout_num / vault.payout_den) - position.claimed

Effects:
  → Transfer entitled from vault to user (CPI)
  → Update position.claimed += entitled
  → Update vault.total_claimed += entitled
  → Emit ClaimEvent
```

**6. close_vault()**

```
Input:
  • authority (signer)
  • vault (mut, close)
  • vault_token_account (close)

Validation:
  ✓ signer == vault.authority
  ✓ vault_token_account.amount <= MAX_DUST_AMOUNT (1000)

Effects:
  → Close vault PDA (reclaim rent)
  → Close vault token account (reclaim rent)
  → Emit VaultClosed event
```

---

### Layer 2: Backend (Indexing & Caching)

**Location:** `backend/`

#### System Components

```
┌──────────────────────────────────────────────────────────┐
│                   Helius Webhooks                        │
│         (Transaction Stream from Solana)                 │
└─────────────────────────┬────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────┐
│          POST /api/webhooks/helius?token={secret}        │
│                                                           │
│  1. Verify authentication                                │
│  2. Parse transaction logs                               │
│  3. Extract account updates (Vault/Position)             │
│  4. Decode with Anchor BorshCoder                        │
│  5. Normalize to DTOs                                    │
│  6. Store in Redis                                       │
│  7. Update indexes (ZSET by authority/owner)             │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────┐
│                  Redis (Vercel KV)                       │
│                                                           │
│  Strings:                                                │
│    • vault:{pda}:json → VaultDTO                         │
│    • position:{pda}:json → PositionDTO                   │
│    • activity:{sig}:{type}:{slot} → ActivityDTO          │
│                                                           │
│  Sets (fallback):                                        │
│    • authority:{pk}:vaults → Set<vaultPda>               │
│    • owner:{pk}:positions → Set<positionPda>             │
│                                                           │
│  Sorted Sets (indexed queries):                          │
│    • authority:{pk}:vaults:by_updated → ZSET<updatedAt>  │
│    • authority:{pk}:vaults:by_updated:{status}           │
│    • owner:{pk}:positions:by_updated                     │
│    • vault:{pda}:activity                                │
│    • owner:{pk}:activity                                 │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────┐
│                    REST APIs                             │
│                                                           │
│  GET /api/vaults?authority={pk}&status={}&cursor={}      │
│  GET /api/positions?owner={pk}&cursor={}                 │
│  GET /api/activity?vault={pda}|owner={pk}&cursor={}      │
│  GET /api/health                                         │
│                                                           │
│  Features:                                               │
│    • Cursor-based pagination (ZSET scores)               │
│    • ETag caching (304 Not Modified)                     │
│    • Status filtering (Funding/Active/Matured/etc)       │
│    • CORS headers                                        │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
                     [Frontend App]
```

#### Data Flow

**Webhook Processing:**

```
1. Helius detects transaction
2. POST to /api/webhooks/helius
3. Extract accountData from transaction
4. Decode using Anchor IDL:
   - BorshCoder.accounts.decode("vault", buffer) → Vault
   - BorshCoder.accounts.decode("position", buffer) → Position
5. Normalize to DTO:
   - Convert BN to string (for JSON safety)
   - Add metadata (slot, updatedAt, updatedAtEpoch)
6. Redis pipeline:
   - SET vault:{pda}:json {json}
   - ZADD authority:{pk}:vaults:by_updated {epoch} {pda}
   - ZADD authority:{pk}:vaults:by_updated:{status} {epoch} {pda}
   - SADD authority:{pk}:vaults {pda}
7. Return { ok: true, processed: {...} }
```

**API Query (GET /api/vaults):**

```
1. Parse query: authority, status, cursor, limit
2. Validate with Zod schema
3. Try ZSET query:
   - Key: authority:{pk}:vaults:by_updated:{status}
   - ZREVRANGE {key} {cursor} {cursor+limit}
   - Scores = updatedAtEpoch
4. Fallback to SET if ZSET empty:
   - SMEMBERS authority:{pk}:vaults
   - Filter by status in-memory
5. Fetch vault JSON:
   - MGET vault:{pda1}:json vault:{pda2}:json ...
6. Generate ETag from JSON + timestamp
7. Check If-None-Match header
8. Return 304 if match, else 200 with data
```

#### Caching Strategy

**Redis TTL:**

- Vault/Position data: No expiration (long-lived)
- Activity events: 30 days (configurable)
- Indexes (ZSET): No expiration

**HTTP Caching:**

- ETag: MD5(JSON + timestamp)
- Cache-Control: public, max-age=30
- 304 Not Modified when ETag matches

**Client-Side (Frontend):**

- localStorage fallback for 304 responses
- React Query staleTime: 30s
- Aggressive polling after mutations (2s for 45s)

---

### Layer 3: Frontend (User Interface)

**Location:** `app/`

#### Component Hierarchy

```
┌────────────────────────────────────────────────────────┐
│                    app/layout.tsx                      │
│  • WalletProvider (Solana wallet adapter)             │
│  • QueryClientProvider (React Query)                  │
│  • SidebarProvider (UI state)                         │
│  • VaultProgramProvider (Anchor Program)              │
└────────────────────────┬───────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
┌──────────────┐  ┌─────────────┐  ┌─────────────┐
│  app/page    │  │ app/        │  │ app/vault/  │
│  (Home)      │  │ portfolio   │  │ [id]/       │
│              │  │             │  │ transparency│
│ • Vault List │  │ • Positions │  │             │
│ • Deposit UI │  │ • Timeline  │  │ • Facts     │
│ • Activity   │  │ • Claim UI  │  │ • Collateral│
│              │  │ • Activity  │  │ • Hedges    │
└──────┬───────┘  └──────┬──────┘  └──────┬──────┘
       │                 │                 │
       │                 │                 │
       ▼                 ▼                 ▼
┌────────────────────────────────────────────────────────┐
│               Custom Hooks Layer                       │
│                                                         │
│  Data Hooks (React Query):                            │
│    • useVaultsAPI() → Backend GET /api/vaults         │
│    • usePositionsAPI() → Backend GET /api/positions   │
│    • useActivityAPI() → Backend GET /api/activity     │
│    • usePortfolioAPI() → Combines vaults + positions  │
│                                                         │
│  Mutation Hooks (Anchor):                             │
│    • useDeposit() → vault.deposit() + polling         │
│    • useClaim() → vault.claim() + polling             │
│    • useMatureVault() → vault.matureVault()           │
│                                                         │
│  Wallet Hooks:                                         │
│    • useVaultClient() → Anchor Program instance       │
│    • useConnection() → Solana RPC connection          │
│    • useWallet() → Connected wallet state             │
└────────────────────────┬───────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
┌──────────────┐  ┌─────────────┐  ┌─────────────┐
│ Backend API  │  │   Solana    │  │  Browser    │
│              │  │  Blockchain │  │  Storage    │
│ • REST calls │  │ • Txs       │  │ • ETag cache│
│ • ETag cache │  │ • Confirms  │  │ • Settings  │
└──────────────┘  └─────────────┘  └─────────────┘
```

#### State Management

**React Query (Server State):**

```typescript
// Query Keys
["vaults", authority, status]
["positions", owner]
["activity", { vault, owner }]
["portfolio", owner, authority]

// Query Options
{
  staleTime: 30_000,        // Fresh for 30s
  gcTime: 5 * 60_000,       // Cache for 5 min
  refetchOnWindowFocus: false,
  retry: smartRetry,        // Only 5xx errors
}

// Mutation Flow
useMutation({
  mutationFn: async () => {
    // 1. Build transaction
    const tx = await program.methods.deposit(amount)...
    // 2. Sign with wallet
    const sig = await wallet.sendTransaction(tx)
    // 3. Confirm
    await connection.confirmTransaction(sig)
    return sig
  },
  onSuccess: () => {
    // 4. Invalidate queries
    queryClient.invalidateQueries(["vaults"])
    // 5. Aggressive polling (2s for 45s)
    startPolling(["vaults", "positions"])
  }
})
```
