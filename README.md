<p align="center"><img src="assets/arcstockpad-logo.png" width="120" alt="ArcStockpad"/></p>

# ArcStockpad

**Launch a coin. Pair it with a real stock.** A launchpad on **Arc mainnet (chain ID 5042)** where every coin is paired with USDC or an issuer-backed tokenized asset — NVDA, AAPL, CRCL, GME, SPY, XAUM gold, cirBTC, EURC, WETH and more.

- App: https://arcstockpad.com
- X: https://x.com/ARCSTOCKPAD
- Docs: https://arcstockpad.com/docs

This repository is the public reference for **integrators** (terminals, trackers, bots) and for anyone who wants to check what is deployed. It contains addresses, ABIs and the integration guide. The application code is not published here.

## How it works

- Every launch is a **real Uniswap V4 pool from block one**, so DEX trackers and routers see an ordinary pool and trades are ordinary `Swap` events.
- The whole 1,000,000,000 supply sits in **one hook-owned, permanently locked position**. The hook has no withdraw function; liquidity is locked from the first block.
- Starting FDV is 2,500–3,500 USDC. Reaching **17,000 USDC FDV** marks the coin as *bonded*: a one-way onchain flag plus a `Graduated` event. Nothing is moved or re-priced at that moment, so a buy of any size fills along the same curve.
- The creator picks the pool fee. 80% goes to the launch's reward mode — **creator rewards**, **holder rewards** or **buyback & burn** — and 20% to the protocol treasury, paid automatically on every swap.
- Gas on Arc is paid in USDC.

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
