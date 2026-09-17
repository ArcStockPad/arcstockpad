# Security

## Reporting an issue

If you believe you have found a vulnerability in the ArcStockpad contracts or website, please report it privately before disclosing it publicly:

- Direct message on X: https://x.com/ARCSTOCKPAD
- Or open a GitHub security advisory on this repository ("Security" → "Report a vulnerability").

Please include the affected address or URL, the steps to reproduce, and the impact you expect. Do not test against other users' funds.

## What is in scope

- The launchpad contracts listed in [`addresses.json`](addresses.json) (source-verified on https://explorer.arc.io).
- https://arcstockpad.com

## Facts worth knowing

- Launch liquidity is a hook-owned Uniswap V4 position with no withdraw path; it is locked from the first block.
- The website never holds keys. Every transaction is signed in the user's own wallet.
- The contracts have not been audited by a third party. Nothing in this repository should be read as an audit claim.
