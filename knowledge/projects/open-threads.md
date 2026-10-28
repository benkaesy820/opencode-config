## Open threads

- The OpenCode TUI re-probes the terminal background only on focus-in, so switching the OS theme needs a click back into the window.
- Superseded 2026-10-21: the read/eval paths in browser-bridge now have contract tests and a node budget (commit 9580e5e).
- 2026-10-28: **republicmart-rust** — added `GET /seller/earnings` in `backend/crates/api/src/seller.rs`: per order the reference, status, `created_at`, item subtotal, `payout_items` paid-out and ledger-derived owed (the seller account's balance for that order, negated), plus totals over the newest 50; a buyer gets `403 not_a_seller`, it is read-only and publishes no event. Commit `c1584ef`, pushed to origin main; `make check` exit 0 (263 tests with every feature, 111 with no database).
