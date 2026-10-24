# handl

**App:** https://handl.finance

handl lets you send **USDG** and **ETH** on **Robinhood Chain** to anyone by their X (Twitter) handle
instead of a wallet address. Sign in with X, get a wallet, and pay `@someone`.

This repository documents how handl works. The source code is kept private.

| Doc | What's in it |
| --- | --- |
| [ARCHITECTURE.md](ARCHITECTURE.md) | How sign-in, wallets, sends and swaps work |
| [API.md](API.md) | The endpoints the app talks to |
| [ON-CHAIN.md](ON-CHAIN.md) | The chain and contracts handl uses |

## How it works

1. **Sign in with X.** Your handle becomes your payment address.
2. **Get a wallet.** Every account has its own wallet on Robinhood Chain. Keys stay on the server,
   encrypted, and never reach the browser.
3. **Send to a handle.** Pay `@anyone` in USDG or ETH.
4. **Deposit, withdraw, swap.** Move funds in and out to any address, and swap between USDG and ETH.
