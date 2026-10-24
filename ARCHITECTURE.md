# Architecture

```
 Browser                        handl backend                    Robinhood Chain
 ───────                        ─────────────                    ───────────────
 landing page + app     ──►     X sign-in, sessions       ──►    USDG / ETH balances
                                per-user wallets                  signed transfers
                                transfers, swaps
```

## Sign-in

X sign-in creates the account. The session is a secure cookie; the browser never holds a key.

## Wallets

Each user gets their own custodial wallet when they sign up. Private keys stay server-side and are
encrypted at rest.

## Balances

Balances are read live from the chain for USDG and ETH, so what the app shows is what the chain holds.

## Sends and withdrawals

The backend builds each transfer, has the user's wallet sign it and broadcasts it to Robinhood Chain.
Before broadcasting it refuses if the balance cannot be read, if there is not enough of the asset, or if
there is not enough ETH left to pay gas. Each transfer has a ceiling, and a form submitted twice with the
same request ID returns the first transaction instead of paying twice.

Transfers are ordinary public transfers, visible to anyone reading the chain.

## Swaps

USDG ⇄ ETH swaps are quoted live and routed through ChangeNOW, which lists both assets on Robinhood
Chain.
