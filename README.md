# handl

**App:** https://handl.finance · **Chain:** Robinhood Chain (4663)

handl is peer-to-peer payments on **Robinhood Chain** where your X (Twitter) handle is the address. Sign
in with X, get a wallet, and send **USDG** or **ETH** to `@anyone`. No copying 42-character hex strings,
no asking for an address first. Settlement is real and on-chain.

> This repository is the public technical overview. The frontend and backend are closed source.

| Doc | Contents |
| --- | --- |
| [ARCHITECTURE.md](ARCHITECTURE.md) | System diagram and how each part behaves |
| [API.md](API.md) | The HTTP surface the app talks to |
| [ON-CHAIN.md](ON-CHAIN.md) | Chain, RPC and contract addresses |

---

## Stack

| Layer | Tech |
| --- | --- |
| Frontend | SvelteKit 2, Svelte 5 (runes), Vite 6, SCSS, Lenis smooth scroll |
| Backend | Node.js HTTP server, no framework, zero runtime dependencies beyond the wallet SDK |
| Wallets | Coinbase Developer Platform (CDP) server wallets, one per user |
| Chain | Robinhood Chain mainnet (chain ID 4663): native ETH + USDG (ERC-20) |
| Swaps | ChangeNOW (USDG ⇄ ETH on Robinhood Chain) |
| Auth | Sign in with X, HttpOnly session cookies |
| Hosting | Vercel (frontend, edge) · Railway (backend, persistent volume) |

The frontend proxies `/api/slide/*` to the backend through Vercel rewrites, so the browser only ever
talks to one origin. That means no CORS surface and the session cookie stays first-party.

## How a payment works

```
 @alice signs in ──► handl creates her CDP wallet
 @alice pays @bob 25 USDG
   │
   ├─ resolve @bob → his handl wallet (he must have signed in once)
   ├─ read alice's USDG + ETH balance live from chain 4663
   ├─ preflight: balance readable? enough USDG? enough ETH left for gas? under the cap?
   ├─ build the ERC-20 transfer, sign it with alice's CDP wallet
   ├─ broadcast the raw transaction to the Robinhood Chain RPC
   └─ return the tx hash; the payment record is keyed by the client's idempotency key
```

## Design decisions

### Custodial wallets, keys never in the browser

Each account gets its own CDP server wallet at sign-up. Private keys live in Coinbase's infrastructure
and are never exposed to handl's frontend or to the user's browser. The backend asks CDP to sign; it
never handles raw key material itself.

### Sign with CDP, broadcast ourselves

CDP's high-level `transfer()` does not know Robinhood Chain. Instead of waiting on support, handl builds
each transaction itself (nonce, gas price, chain ID, calldata), has CDP `signTransaction()` it, and relays
the signed raw transaction straight to the chain's RPC. That works on any EVM chain CDP can sign for.

### Chain is the source of truth

There is no internal balance ledger to drift out of sync. Balances are read live from chain 4663 on
every request, so what the app shows is what the chain holds. At boot the server checks the RPC's
`eth_chainId` against its configured chain and refuses to run against the wrong network, because a
mismatched RPC would make real deposits read as zero.

### Fail closed before broadcasting

Every send runs preflight guards and refuses before anything touches the chain if:

- the balance cannot be read (an RPC hiccup is never treated as "probably fine"),
- there is not enough of the asset,
- there is not enough ETH left to pay for the transaction's own gas,
- the amount exceeds the per-transfer ceiling,
- the destination is the zero address (funds would be burned) or the sender's own wallet.

### Idempotent writes

Every payment, withdrawal and swap carries a client-generated idempotency key. A resubmitted form, a
double tap or a retried request returns the original transaction instead of paying twice.

### Sessions

Sessions are random 192-bit IDs in an HttpOnly, SameSite=Lax cookie with a 30-day lifetime. A single
server-side timestamp can revoke every session issued before it, and secrets are compared in constant
time.

### Swaps without a DEX

There is no canonical AMM for USDG/ETH on Robinhood Chain yet, so swaps route through ChangeNOW, which
lists both assets natively on the chain. Quotes are fetched live and the exchange deposit address is
paid from the user's wallet like any other send.

## Frontend

- Landing page and app share one wordmark and design language; the app runs in its own shell at `/app`.
- Svelte 5 runes for state, SCSS design tokens, Lenis-driven scroll animations on the landing page.
- Mobile-first layout, tested on narrow devices so that the shell never overflows the viewport.
- Every user has a public profile at `handl.finance/@handle`, served through the same rewrite.

## Privacy note

Transfers are ordinary public transactions on Robinhood Chain: not shielded and not anonymised. Anyone
reading the chain can see them, and the app says so.
