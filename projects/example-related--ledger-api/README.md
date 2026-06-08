# ledger-api — Harness Brain index

> **Scenario: related repos** — one product, multiple repositories.
> This folder is part of the **Ledger** product, which spans two repos tracked
> in this brain. The `example-related--` prefix is an illustrative label; a real
> folder is just the repo name (`ledger-api`).

**Part of:** Ledger (product)
**Sibling repos (related):**
- `example-related--ledger-web` — the web client; **depends on** this API.

Because these repos are related, the `cross-repo-discovery-agent` reads **both**
Ledger folders together at session start and builds a single cross-repo digest:
a change here (e.g. an API contract change) surfaces when you open `ledger-web`.

| Date | Entries | File |
| --- | --- | --- |
| 2026-Jun-07 | 1 | [2026-Jun-07.har.compact.md](2026-Jun-07.har.compact.md) |
