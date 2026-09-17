<p align="center"><img src="assets/arcstockpad-logo.png" width="120" alt="ArcStockpad"/></p>

# ArcStockpad

**Launch a coin. Pair it with a real stock.** A launchpad on **Arc mainnet (chain ID 5042)** where every coin is paired with USDC or an issuer-backed tokenized asset — NVDA, AAPL, CRCL, GME, SPY, XAUM gold, cirBTC, EURC, WETH and more.

- App: https://arcstockpad.com
- X: https://x.com/ARCSTOCKPAD
- Telegram: https://t.me/arcstockpad
- Docs: https://arcstockpad.com/docs

> **Official $ASPAD contract:** `0x3Dd6Db0F20D747e1274839Cdd6724Cc5Df1fb813` — verify it against [`addresses.json`](addresses.json) or [explorer.arc.io](https://explorer.arc.io/address/0x3Dd6Db0F20D747e1274839Cdd6724Cc5Df1fb813) before trusting any other address.

This repository is the public reference for **integrators** (terminals, trackers, bots) and for anyone who wants to check what is deployed. It contains addresses, ABIs and the integration guide. The application code is not published here.

## How it works

- Every launch is a **real Uniswap V4 pool from block one**, so DEX trackers and routers see an ordinary pool and trades are ordinary `Swap` events.
- The whole 1,000,000,000 supply sits in **one hook-owned, permanently locked position**. The hook has no withdraw function; liquidity is locked from the first block.
- Starting FDV is 2,500–3,500 USDC. Reaching **17,000 USDC FDV** marks the coin as *bonded*: a one-way onchain flag plus a `Graduated` event. Nothing is moved or re-priced at that moment, so a buy of any size fills along the same curve.
- The creator picks the pool fee. 80% goes to the launch's reward mode — **creator rewards**, **holder rewards** or **buyback & burn** — and 20% to the protocol treasury, paid automatically on every swap.
- Gas on Arc is paid in USDC.

## Safety — check it yourself

Every point below can be confirmed from the verified source and live state on https://explorer.arc.io.

- **Verified source.** The hook, the factory and every launched token are full-match verified on the Arc explorer.
- **Liquidity is locked from the first block.** Live, browser-side proof for any coin: https://arcstockpad.com/proof/&lt;token&gt; — it reads the position from Uniswap V4 StateView and shows the locked share, both sides of the position and the exact eth_call to repeat. The launch position belongs to the hook. The hook adds liquidity once, at launch, and afterwards only collects swap fees; it has no function that decreases or withdraws that liquidity, and no LP token exists. Before a coin is bonded nobody else can add liquidity to the pool. To check a coin: Uniswap V4 StateView `getPositionInfo(poolId, hook, tickLower, tickUpper, 0x0)` returns the same liquidity as `hook.launches(poolId).curveLiquidity`.
- **Fixed supply.** A token mints 1,000,000,000 units once, in its constructor. There is no mint function, no owner, no pause, no blacklist and no transfer tax.
- **The fee cannot change.** The creator picks 0.1%–10% at launch; it is part of the immutable Uniswap V4 pool key.
- **Trading cannot be paused.** The hook has no pause switch. Swaps go straight through the Uniswap V4 PoolManager with any V4 router.
- **What the admin can do.** Pause *new launches* on the factory, change the launch fee (hard cap 50 USDC), and approve quote assets and price sources used when a *new* coin is created. Factory ownership is two-step. The treasury can claim the 20% protocol share. None of this can move a coin's liquidity, change its fee or stop its trading.
- **No custody.** The website never holds keys; every transaction is signed in the user's own wallet.

The contracts have not been audited by a third party.

## Contracts

All source-verified on https://explorer.arc.io. Machine-readable copy: [`addresses.json`](addresses.json).

| Contract | Address |
|---|---|
| Launch hook — **curve-v2.4** (current) | [`0xe92F7b4362EDEd62Ed7A10Ac90262B0bF4552840`](https://explorer.arc.io/address/0xe92F7b4362EDEd62Ed7A10Ac90262B0bF4552840) |
| Launch factory — curve-v2.4 (current) | [`0x79Cc46Bf4F5c1cC42e81259E957BCF9a9D3b08ea`](https://explorer.arc.io/address/0x79Cc46Bf4F5c1cC42e81259E957BCF9a9D3b08ea) |
| Legacy hook / factory — curve-v2.3 | `0x73FaA75815D3239834262aac97F6C5b51efE2840` / `0x61dE1811f521f7A76A520c843569586f4782BFFa` |
| Legacy hook / factory — curve-v2.2 | `0xcfBf5dBA3452dE0DF43CAC899ed74393e811A840` / `0x37012298CeF699BDf7057c5B22Ec9dc075A1F651` |
| Stock asset registry | `0xf6EA18321A378C69EB4Fd3618db0f76D35f4EdF5` |
| USD valuation policy | `0x6814D60653ec70e6137Fffe42Dc8BE87F3bf9F28` |
| Uniswap V4 PoolManager (Arc) | `0x8366a39cc670b4001a1121b8f6a443a643e40951` |

Legacy builds keep serving the coins launched on them. All builds share the same events, selectors and `launches()` layout, so one adapter covers every address above.

**Live example — $ASPAD** (launched and bonded on curve-v2.4): token [`0x3Dd6Db0F20D747e1274839Cdd6724Cc5Df1fb813`](https://explorer.arc.io/address/0x3Dd6Db0F20D747e1274839Cdd6724Cc5Df1fb813) · [chart](https://arcstockpad.com/token/0x3Dd6Db0F20D747e1274839Cdd6724Cc5Df1fb813) · [DexScreener](https://dexscreener.com/arc/0x62048ba5a1ae633e1b2aef70a5261db53799ecf2e5754d680ed89d9c8e91c31d) — USDC pair, creator-rewards mode, real Uniswap V4 pool from launch.

## Token Locker

A permissionless ERC-20 timelock at https://arcstockpad.com/locker — lock any token on Arc until a date you choose and share the public proof page `arcstockpad.com/locker/<token>`.

| Contract | Address |
|---|---|
| ArcTokenLocker | [`0x02C6C80E26198Eb4669CDF409d756d6E9D6D895e`](https://explorer.arc.io/address/0x02C6C80E26198Eb4669CDF409d756d6E9D6D895e?tab=contract) — verified |
| ArcLockVault (implementation cloned once per lock) | [`0x2badFB974F49492eb1F6023F91FEed7A5CB5d6B5`](https://explorer.arc.io/address/0x2badFB974F49492eb1F6023F91FEed7A5CB5d6B5?tab=contract) — verified |

- **No owner, no fee, no pause, no upgrade, no early exit.** Nobody — not the depositor, not ArcStockpad — can move a locked balance before its unlock time.
- **Extend only.** A lock's date can be pushed later, never earlier.
- **One isolated vault per lock.** Locks never share a balance, so nothing that happens to one lock can touch another.
- **Rewards keep flowing.** Anything that arrives at a vault other than the locked token (holder rewards, airdrops) can be collected by the lock's owner at any time; the locked token itself cannot.
- Independent of the launchpad: the hook, factory and launched tokens never call it and it has no privileges over any token.

Read a lock: `getLock(id)` → `(token, owner, vault, unlockAt, createdAt, withdrawn, label)`, `lockedAmount(id)`, `locksForToken(token)`, `locksOf(owner)`. Events: `Locked`, `Extended`, `Added`, `OwnerChanged`, `Withdrawn`, `Swept`. ABIs: [`abi/ArcTokenLocker.json`](abi/ArcTokenLocker.json), [`abi/ArcLockVault.json`](abi/ArcLockVault.json).

## For integrators

Start with [`docs/integration-guide.md`](docs/integration-guide.md): launch discovery (`LaunchCreated`), pool keys, curve progress, bonded status (`Graduated`), fee accounting, topics and selectors. ABIs are in [`abi/`](abi).

Quick reference:

| What | Where |
|---|---|
| New launches | factory event `LaunchCreated(uint256 id, address token, bytes32 poolId, address quote, address creator)` |
| Launch details | `factory.getLaunch(id)` → `(token, quote, creator, mode, fee, poolId)` |
| Curve / bonded state | `hook.launches(poolId)` → `graduated`, tick band, liquidity |
| Trades | Uniswap V4 PoolManager `Swap` events for the `poolId` |
| Token | fixed-supply ERC-20, 18 decimals, 1,000,000,000 total |

## Pair policy

Only custodial / issuer-backed stock tokens, and commodities only when a real issuer lists the Arc address. No clone or meme tokens named after stocks.

## Disclaimer

Nothing here is financial advice or a claim of an audit. Bonding-curve tokens are highly volatile and can lose all value. See [SECURITY.md](SECURITY.md) to report an issue.
