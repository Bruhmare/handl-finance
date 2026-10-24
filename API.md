# API

All routes live under `/api/slide/` and need a signed-in session unless marked public.

| Area | Routes |
| --- | --- |
| Sign-in | `auth/x/login`, `auth/x/callback`, `auth/logout` |
| Account | `users/me`, `users/me/balances`, `users/me/address` |
| Lookup (public) | `users/resolve/:handle`, `public/users/:handle` |
| Payments | `payments`, `payments/:id`, `transactions`, `assets` |
| Moving funds | `deposit`, `withdraw` |
| Swaps | `swap/quote`, `swap` |
| Chain | `chain/status`, `chain/balance` |
