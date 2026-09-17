# ArcStockpad — integration guide for terminals, trackers and bots

Everything below can be checked on https://explorer.arc.io. Nothing here claims an audit.

## Overview

- Every ArcStockpad launch is a **real Uniswap V4 pool from block one** (the whole supply is one hook-owned, permanently locked position), so existing V4 indexers already see trades. This guide covers what an integrator needs on top of that: the **launchpad label, curve progress and "bonded / migrated" status**.
- Contracts are **verified on the Arc explorer** (full source match): factory, hook, bootstrap, registry, valuation policy, price adapters and the token implementation.
- Multiple tokens have already gone through the full lifecycle on mainnet (launch → curve → bonded at 17,000 USDC FDV), including $ASPAD on the current v2.4 build. **All hook builds share the same events, selectors and `launches()` layout**, so one adapter covers every address in the table below.

## Chain and addresses (Arc mainnet, chain ID 5042)

| Item | Value |
|---|---|
| RPC / explorer | https://rpc.mainnet.arc.io · https://explorer.arc.io |
| Site | https://arcstockpad.com |
| **Launch factory (current, v2.4)** | `0x79Cc46Bf4F5c1cC42e81259E957BCF9a9D3b08ea` — verified |
| **Launch hook (current, v2.4)** | `0xe92F7b4362EDEd62Ed7A10Ac90262B0bF4552840` — verified; V4 flags `0x2840` (beforeInitialize · beforeAddLiquidity · afterSwap); `BUILD()` returns `"curve-v2.4"` |
| Legacy factory / hook (v2.3, 2 launches, still live) | `0x61dE1811f521f7A76A520c843569586f4782BFFa` / `0x73FaA75815D3239834262aac97F6C5b51efE2840` — verified |
| Legacy factory / hook (v2.2, 1 launch, still live) | `0x37012298CeF699BDf7057c5B22Ec9dc075A1F651` / `0xcfBf5dBA3452dE0DF43CAC899ed74393e811A840` |
| Uniswap V4 PoolManager | `0x8366a39cc670b4001a1121b8f6a443a643e40951` |
| Uniswap V4 StateView | `0xf3334192d15450cdd385c8b70e03f9a6bd9e673b` |
| Curve swapper (generic V4 router used by the site) | `0x83cc33fD588A206c27f6B5c26F13Af3e7231e31D` |
| Token implementation | ArcRewardToken (ERC-20, 1,000,000,000 fixed supply, 18 dp) — e.g. NOVA `0x7a6239e50274C70D7f2BBA43A6d9d100752e37c0` (verified) |
| Quote assets | USDC `0x3600000000000000000000000000000000000000` (6 dp) plus registry-approved stock tokens (NVDA, CRCL, GME, SPY…) and crypto (cirBTC, EURC, WETH — EURC/WETH priced on Aerodrome Slipstream TWAPs) |

Worked example — **$ASPAD / USDC** (launched and bonded on the current v2.4 hook):
- token `0x3Dd6Db0F20D747e1274839Cdd6724Cc5Df1fb813`, poolId `0x62048ba5a1ae633e1b2aef70a5261db53799ecf2e5754d680ed89d9c8e91c31d`
- launch tx `0xc87ed908f77689b5b488c8008e23c18ecf0ecbd5ee504f6063fc5e72ff2d637a` (block 21332455), bonded tx `0x13d8fd71ca0ab0dd5188f01ed3f299e3e29c7280e5bee22d460d641bc131dca4` (block 21334830)
- DexScreener already indexes it as Uniswap V4: https://dexscreener.com/arc/0x62048ba5a1ae633e1b2aef70a5261db53799ecf2e5754d680ed89d9c8e91c31d

Earlier example — **NOVA / NVDA** (launched on the legacy v2.2 hook, graduated 2026-09-16):
- token `0x7a6239e50274C70D7f2BBA43A6d9d100752e37c0`, poolId `0x5e986747844a281809f17843e453ce4ba79bdf98446701fb2135beac2048cff5`
- launch tx `0xcdf96361…b958` (block ~21170766), graduation tx `0x4515ae53c70f855ff55d97834f0415aae14db8f8a14bcdc904eb2f35b0d1e401` (block 21174509)
- DexScreener already indexes it as Uniswap V4: https://dexscreener.com/arc/0x5e986747844a281809f17843e453ce4ba79bdf98446701fb2135beac2048cff5

## How to detect and classify our launches

1. **Discover launches** from the factory event (indexed: id, token, poolId):
   `LaunchCreated(uint256 id, address token, bytes32 poolId, address quote, address creator)` — topic0 `0xfb49a27bdef3a51e9524da38397dfcf205e9fc95d9dddb60b58fd456e350913b`
   (`getLaunch(uint256)` `0x5930d3ce` returns `(token, quote, creator, mode, fee, poolId)`; `launchCount()` `0x27cca59f`; `tokenIndex(address)` `0x427f91a6` is index+1; metadata: `logoURI(uint256)` `0x7310fbc7`, `website(uint256)` `0x09076472`, `twitter(uint256)` `0x58fe3f39`.)
2. **Pool key**: `hook.poolKey(bytes32 poolId)` `0xca266dfe` → `(currency0, currency1, fee, tickSpacing = 10, hooks = hook)`. poolId is the standard V4 `keccak256(abi.encode(PoolKey))`.
3. **Curve state**: `hook.launches(bytes32 poolId)` `0xad091230` → `(token, quote, creator, mode, tokenIs0, graduated, lower, upper, curveLiquidity, lpLiquidity, reservedTokens, lockedTokenDust, lockedQuoteDust)`.
   - **On the curve / not bonded yet**: `graduated == false`. Progress = position of the pool's current tick (StateView `getSlot0(poolId)`) between the start tick and the graduation tick: if `tokenIs0` the curve runs from `lower` (start) up to `upper` (graduation); otherwise from `upper` (start) down to `lower` (graduation).
   - **Bonded / migrated**: `graduated == true`. On the current v2.4 hook this is a **one-way label**: the pool keeps the same single locked position (start tick → usable tick limit, no withdraw function), nothing is moved or re-priced, the price can later trade back below the threshold, and the flag stays true. On the legacy v2.2/v2.3 hooks the curve is converted into a permanent full-range position at that moment. Event: `Graduated(bytes32 poolId, uint160 sqrtPrice, uint128 liquidity, uint256 tokensLocked, uint256 quoteLocked)` — topic0 `0x144305682787fdd32c87b69bf6caf17b45ba4e902bd1e7e6dae5200722e0f087`, emitted inside the crossing swap. Graduation threshold: 17,000 USDC FDV (fixed in ticks at launch; start FDV 2,500–3,500 USDC).
   - Launch event on the hook: `Launched(bytes32 poolId, address token, address quote, int24 lower, int24 upper, uint128 curveLiquidity, uint256 curveTokens, uint256 reservedTokens)` — topic0 `0xc18c9c78400fd5a28d5c28ea31385fc4276f2ea0c571d3a051baec3ca1967171`.
4. **Trades** are ordinary PoolManager `Swap` events on the poolId — nothing custom. Buys/sells route through any V4 router (UniversalRouter, our swapper, yours). Pool LP fee is creator-chosen (0.1%–10%); 80% goes to the launch's reward mode, 20% to the protocol treasury, paid automatically inside `afterSwap`. Fee accounting: `hook.fees(bytes32)` `0xcdb5661f` → `(protocolOwed, creatorOwed, holderPending, buybackPending, pendingTokenFees)`.
5. **Dev/creator**: `creator` from `getLaunch`; the creator wallet receives fee payouts as quote-token transfers from the hook.
6. Chart note: on v2.4 a buy of any size that crosses the threshold is one ordinary `Swap` that fills along the same curve (no hook-originated swap, no tick-limit print, no wick). v2.3 launches bond through a thin 2% backstop plus one real-amount "park" swap whose sender is the hook; v2.2 (NOVA) predates both.
7. Liquidity lock: on v2.4 the position is hook-owned and locked from block one; `lpLiquidity == curveLiquidity` once bonded. There is no LP token and no withdraw path in any build.

## Verifying the liquidity lock (for "LP locked" badges)

Checks that look for LP tokens in a burn address or in a third-party locker return nothing here, because Uniswap V4 positions are not tokens. Read the position instead:

```
StateView 0xf3334192d15450cdd385c8b70e03f9a6bd9e673b
getPositionInfo(poolId, owner = the hook, tickLower, tickUpper, salt = 0x0) -> liquidity
getLiquidity(poolId)                                                        -> the pool's active liquidity
```

Tick range to pass:

| Build | tickLower | tickUpper |
|---|---|---|
| curve-v2.4 (current) | `launches(poolId).lower` if `tokenIs0`, else `-887270` | `887270` if `tokenIs0`, else `launches(poolId).upper` |
| v2.2 / v2.3, bonded | `-887270` | `887270` |
| v2.2 / v2.3, on the curve | `launches(poolId).lower` | `launches(poolId).upper` |

The hook-owned share of `getLiquidity(poolId)` is the locked share; anything above it was added by outside LPs after bonding and is theirs to withdraw. The hook cannot withdraw its own position: in the verified source `modifyLiquidity` is called once at launch with a positive delta and afterwards only with a delta of `0` (fee collection), and there is no owner, admin, pause or upgrade path.

A live, browser-side version of this check for any coin: `https://arcstockpad.com/proof/<token>`.

## Token locker (optional, separate contract)

Creators can also timelock their own tokens in `ArcTokenLocker` [`0x02C6C80E26198Eb4669CDF409d756d6E9D6D895e`](https://explorer.arc.io/address/0x02C6C80E26198Eb4669CDF409d756d6E9D6D895e?tab=contract) — no owner, no fee, extend-only, one vault per lock. Read `locksForToken(token)` → ids, then `getLock(id)` → `(token, owner, vault, unlockAt, createdAt, withdrawn, label)` and `lockedAmount(id)`. Events: `Locked`, `Extended`, `Added`, `OwnerChanged`, `Withdrawn`, `Swept`. Public page per lock: `https://arcstockpad.com/locker/lock/<id>`.

ABIs are in [`/abi`](../abi). Contract sources are verified and readable on the Arc explorer at the addresses above.
