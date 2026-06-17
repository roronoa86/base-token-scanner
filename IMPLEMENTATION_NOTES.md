# Implementation Notes - BASE FORENSICS v4

## Core Issues Resolved

### Detection Logic
✅ **Clanker Detection Multi-Layer System**
- Factory address check via Basescan
- allData() function check via RPC  
- isVerified() function check via RPC
- ABI inspection for Clanker function signatures
- Context field parsing from on-chain data

### Data Fetching Resilience
✅ **Multi-Fallback API System**
- Primary API → allorigins.win proxy → corsproxy.io proxy
- Blockscout v2 → Blockscout v1 → Basescan
- All requests have 7-10 second timeouts
- Graceful degradation when data unavailable

### Risk Assessment Accuracy
✅ **Corrected Scoring Rules**

**Clanker Tokens (isVerified=true)**
- LP in Uniswap v4 PoolManager (Clanker-managed)
- Admin CANNOT remove LP → automatic RUG SAFETY
- Still need to check: honeypot, high tax, holder concentration
- Score impact: -5 to 0 (reduces risk due to safe LP)

**Clanker Tokens (isVerified=false)**
- Deployed via 3rd-party (Bankr, WarpCast, etc.)
- LP NOT automatically managed by Clanker
- Must check: LP lock status, deployer, holder concentration
- Score impact: +1 (yellow flag, requires manual LP check)

**Non-Clanker Tokens (Regular ERC-20)**
- Source code verification is useful but NOT critical red flag
- Many legitimate tokens unverified (especially new ones)
- Changed penalty from 2pts (RED) to 1pt (YELLOW)
- Real red flags: honeypot, mint function, blacklist, high tax
- AERO and similar tokens now score correctly

### Holder Analysis
✅ **Fixed Concentration Calculation**
- Correctly filters out Pool Manager address
- Handles multiple holder API response formats
- Accurate top-3 and top-10 percentage calculation
- Identifies smart contracts vs EOAs

## Known Limitations & Workarounds

### API Rate Limiting
- GoPlus Labs API: ~10 requests/minute free tier
- DexScreener: Generous rate limits
- Basescan: Rate limited without API key
- **Solution**: Use proxy fallbacks, add small delays if needed

### Blockscout Holders Endpoint
- Sometimes returns empty for new tokens
- Basescan fallback picks up holders
- New tokens may not have holder data available yet

### LP Lock Detection
- Works well for Uniswap v3/v2 standards
- Mudra, Unicrypt, PinkLock, Team.Finance supported
- Custom lockers may not be detected

### Multichain OFT Tokens
- Currently only analyzes Base network
- For LayerZero tokens, cross-chain minting risk not checked
- Would require reading multiple chain RPCs

## Testing with Real Tokens

### Test Case 1: Legitimate Clanker (Official)
**Expected**: LOW risk
- isVerified = true
- LP in Clanker v4 official pool
- Normal holder distribution

### Test Case 2: Clanker via Bankr
**Expected**: MEDIUM risk (requires LP check)
- isVerified = false
- Deploy via Bankr
- Check if LP locked or not

### Test Case 3: Legitimate Non-Clanker (e.g., AERO)
**Expected**: LOW-MEDIUM risk
- Regular ERC-20
- Verified source OR normal functions
- Good LP/holder distribution
- SHOULD NOT be HIGH just for "unverified"

### Test Case 4: Obvious Scam
**Expected**: HIGH-EXTREME risk
- Honeypot detected via GoPlus
- OR: Mint function + admin
- OR: >50% supply in single holder
- OR: Owner renounced but proxy contract
- OR: Multiple red flags combined

## Performance Optimizations Applied

1. **Parallel API Calls**: Uses Promise.all() for independent requests
2. **Early Exit**: Returns immediately if contract not found
3. **Batch RPC Calls**: Groups multiple reads into single eth_call batch
4. **Selective Data**: Only fetches needed fields from APIs

## Future Improvements

1. **Caching Layer**: Store recent scans to reduce API calls
2. **Historical Trends**: Track holder distribution over time
3. **Machine Learning**: Improve scam detection with patterns
4. **Community Reports**: Integration with ZachXBT / investigator findings
5. **Custom Rule Builder**: Let users create custom risk rules
6. **Mobile Optimization**: Better responsive design for mobile
7. **Dark Mode Toggle**: User preference for theme
8. **Report Export**: PDF/CSV download of analysis

## Debugging Tips

### Check Browser Console
```javascript
// Clanker detection output
[v0] Clanker Detection: {
  creatorFromBasescan,
  creatorFromBlockscout,
  isCreatedViaClankerFactory,
  hasAllData,
  hasIsVerifiedFunction,
  isClanker
}

// RPC probe results
[v0] isVerified probe: {verHex, decoded}

// ABI parsing
[v0] ABI parse error: "message"
```

### Manual Test Function
```javascript
// In browser console:
document.getElementById('addrIn').value = '0x...';
await startScan();
```

### Check API Responses
```javascript
// Test GoPlus
await fetchGoPlus('0x...');

// Test Blockscout
await fetchBSHolders('0x...');

// Test DexScreener
await fetchDex('0x...');
```

## Code Structure

```
index.html
├── CSS Styles (lines 12-125)
├── HTML Structure (lines 127-158)
└── JavaScript (lines 160-1547)
    ├── Constants & Factories (lines 161-210)
    ├── Helper Functions (lines 213-500)
    │   ├── LP Lock Parsing
    │   ├── RPC Batch Helpers
    │   ├── Decode Functions
    │   └── Fetch Wrappers
    ├── Step Manager (lines 244-260)
    ├── Main Scan Function (lines 585-711)
    │   ├── Validation
    │   ├── RPC Reads
    │   ├── API Fetches
    │   ├── Scoring
    │   └── Rendering
    ├── Scoring Engine (lines 716-1004)
    │   ├── Clanker Detection
    │   ├── LP Analysis
    │   ├── Security Scan
    │   ├── Tax Analysis
    │   ├── Holder Distribution
    │   └── Verdict Calculation
    ├── Render Engine (lines 1009-1469)
    │   ├── Verdict Card
    │   ├── Sections (Market, Security, Holders, etc.)
    │   └── Flags Display
    └── Utilities (lines 1474-1547)
        ├── Formatters (price, number, supply)
        └── Event Listeners
```

## Deployment Checklist

- [x] Fix critical bugs (Clanker detection, holders, ABI)
- [x] Fix scoring inaccuracies (non-Clanker penalties)
- [x] Add error handling and logging
- [ ] Load test with multiple concurrent scans
- [ ] Test all token types (Clanker, ERC-20, OFT)
- [ ] Verify scores match manual analysis
- [ ] Performance profile (loading time)
- [ ] Mobile responsiveness check
- [ ] Deploy to production

## Version History

- **v4.0**: Bankr Skill Logic (current)
  - Clanker v4 detection
  - Multi-source API fallbacks
  - Advanced risk scoring
  - Holder concentration analysis
