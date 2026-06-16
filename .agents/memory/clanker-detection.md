---
name: Clanker Detection Logic
description: Correct ABI selectors, isVerified semantics, LP lock detection for Base Forensics app
---

## Correct ABI Selectors
- `allData()` → `0xb974b0a3` (keccak256 of "allData()") — was wrong as 0x9a5fecd3
- `isVerified()` → `0x80007e83` (keccak256 of "isVerified()") — was wrong as 0xbdda81d9
- Both wrong selectors were the root cause of all Clanker detection failures

## isVerified() Semantics (user-confirmed, critical)
- `isVerified = true` → Token deployed via clanker.world (official). LP is inside Uniswap v4 PoolManager managed by Clanker protocol. Admin CANNOT remove LP. SAFE.
- `isVerified = false` → Token deployed via 3rd-party app (Bankr, WarpCast bot, etc.). LP not automatically Clanker-managed. Still a valid Clanker token but LP needs separate check.

**Why:** Clanker's PoolManager controls LP at protocol level for verified tokens. Non-verified = bypassed official UI.

## Source Code Verification for Clanker
Do NOT penalize Clanker tokens for unverified source code. Factory deploys have identical bytecode across all tokens — this is expected per Clanker architecture.

## Clanker Factory Addresses (Base)
- `0xe85a59c628f7d27878aceb4bf3b35733630083a9` = Clanker v4
- `0x2a787b2362021cc3eea3c24c4748a6cd5b687382` = Clanker v3.1
- `0x375686ac453b56c98a49b3cad82a4ca10571d3ef` = Clanker v3.0
- `0x256dddd03b9b94098939763dc0b4d4b732fb6bb1` = Clanker v2/Doppler
- `0x1a0ad19a73752ea416c116c4c2c62c3e414c5b36` = Clanker v1

## LP Lock Detection
Primary source: GoPlus API `lp_holders[]` field — each entry has `locked: 1` flag and `tag` (e.g., "Unicrypt").
- Parse in `parseLPLock(goplus)` function → returns `{hasData, locked, lockedPct, burnedPct, lockerNames}`
- For Clanker verified → skip LP lock check (protocol-managed)
- For Clanker unverified / non-Clanker → LP lock check is critical; no lock = RED FLAG (score +3)

## allData() Response Parsing
- Minimum valid response length: 322 chars (5 head slots × 64 + "0x")
- String decode: use `.replace(/\0/g,'')` NOT `.replace(/ /g,'')` — spaces must be preserved
- Slot layout: [0]=originalAdmin, [1]=admin, [2]=image offset, [3]=metadata offset, [4]=context offset
