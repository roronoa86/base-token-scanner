# Quick Reference: Token Scanner Logic v3

## How isVerified() Detection Works

### Clanker Token Detection Layers (ANY layer = Clanker)
1. **Factory Check**: Created via official Clanker factory (v1-v4)
2. **allData() Present**: Has allData() function (Clanker-specific)
3. **isVerified() Callable**: Has isVerified() function (Clanker-specific)
4. **Context Check**: allData context includes "clanker"
5. **ABI Check**: ABI contains "alldata" and "originaladmin"

### isVerified() Return Value Interpretation
- **TRUE** → `clanker.world` official deployment (GREEN ✓)
- **FALSE + Farcaster context** → Farcaster miniapp Clanker (GREEN ✓)
- **FALSE + no Farcaster** → 3rd-party app like Bankr/Doppler (YELLOW ⚠)
- **null/timeout** → Not a Clanker token (INFO ℹ)

## Risk Scoring Formula

**Base Score: 0-28 points**

### Green Flags (0 points = good, removes flags)
- isVerified = TRUE (official Clanker)
- isVerified = FALSE + Farcaster (official miniapp)
- LP locked ≥80%
- LP burned ≥80%
- LP in Clanker v4 pool (protocol-managed)
- No dangerous functions in ABI
- Source code verified
- Buy/sell tax 0%

### Yellow Flags (1 point each)
- isVerified = FALSE (3rd-party)
- LP partially locked (<80%)
- LP partially burned
- LP lock status unknown
- Proxy contract
- Trading cooldown
- High buy/sell tax (5-10%)
- Creator holding 5-15%
- GoPlus unavailable

### Red Flags (2+ points each)
- No LP pair (3 points)
- LP not locked/burned (3 points)
- Honeypot detected (5 points)
- Mint function (3 points)
- Owner can change balances (4 points)
- Hidden owner (3 points)
- Blacklist function (2 points)
- Low liquidity (2-3 points)

## Verdict Levels

| Score | Verdict | Color | Action |
|-------|---------|-------|--------|
| 0-2   | VERY LOW RISK | Green | Safe for trading |
| 3-5   | LOW-MEDIUM RISK | Yellow-Green | Acceptable risk |
| 6-10  | MEDIUM RISK | Yellow | Caution needed |
| 11-20 | HIGH RISK | Orange | Risky, research more |
| 21-28 | CRITICAL | Red | Likely scam |

## ABI Status Messages

### For Clanker Tokens
- **"ABI tersedia (Clanker)"** → ABI found, can scan functions
- **"ABI tidak fully tersedia"** → Factory bytecode, standard for Clanker

### For Non-Clanker Tokens
- **"ABI tidak tersedia"** → 1 point penalty, can't verify functions

### Source Code Verification
- **Clanker**: Not penalized (factory bytecode standard)
- **Non-Clanker**: 1 point penalty if unverified

## Farcaster Miniapp Support

Token deployed via Farcaster Clanker miniapp is detected by:
1. Has `isVerified()` function (returns FALSE)
2. Has `allData()` with context containing "farcaster"
3. Treated as **official Clanker variant** (same LP safety as clanker.world)

**Result**: No LP lock penalty even though isVerified=FALSE

## LP Lock Status Logic

### Clanker Tokens (All Variants)
- **Official clanker.world** (isVerified=TRUE)
- **Farcaster miniapp** (isVerified=FALSE + Farcaster context)
- **Action**: SKIP LP check (protocol-managed)

### Other Tokens
- **Check GoPlus lp_holders data**
- ≥80% locked/burned = GREEN
- 0-79% = YELLOW
- 0% locked/burned = RED (3 point penalty)

## Scoring Example: Token FDOR

```
Token: 0x3Fa5F99392461aA97Ea25AB38860c985DE28Cb07
Type: Farcaster Clanker Miniapp

Checks:
1. Base RPC ✓ (supply OK)
2. GoPlus ✓ (security OK)
3. DexScreener (no pair yet - new token)
4. Blockscout ✓ (FDOR detected)
5. Top Holders ? (token too new)
6. ABI ✓ (tidak terverifikasi, no penalty)
7. Deployer ✓ (nonce 4512)
8. Risk Scoring → 4 points total

Breakdown:
- isVerified = FALSE (Farcaster) = GREEN ✓ (0 points)
- LP protocol-managed = GREEN ✓ (0 points)
- No DEX pair = RED (2 points)
- Top holders too few = RED (2 points)
- ABI availability = INFO (0 points)
- Source unverified = INFO (0 points)

Final: Score 4/28 (LOW-MEDIUM RISK) ✓
```

## Debugging Tips

### To verify Farcaster detection:
1. Check browser console for: `[v0] Clanker Detection: { isFarcasterClanker: true }`
2. Look for allData() context: "farcaster" or "Farcaster Clanker Miniapp"
3. Verify isVerified() returns FALSE

### To check ABI status:
1. Look at Basescan contract code (may not be verified)
2. Check if ABI contains "alldata" and "originaladmin" functions
3. Verify hasValidAbi = true in console log

### To check LP safety:
1. For Clanker: No LP check needed (protocol-managed)
2. For others: Check GoPlus lp_holders percentage
3. Look for lock contracts (Unicrypt, Team.Finance, etc.)

## Common Questions

**Q: Why does my Farcaster token show score 4 instead of 8?**
A: It's correctly detected as official Clanker variant with protocol-managed LP. Old logic incorrectly penalized it.

**Q: Why doesn't my ABI show as verified?**
A: Clanker factory tokens never get individually verified on Basescan. This is normal and not a penalty.

**Q: Is my Farcaster token safe if score is 4?**
A: Yes, for a fresh token it's LOW-MEDIUM risk. Clanker tokens are protected by protocol (LP locked, no minting, etc.). Monitor: DEX pair setup, holder distribution.

**Q: Why do I get "Token baru?" error on Top Holders?**
A: Token too new (no DexScreener pair yet). This resolves once trading volume occurs.
