# Harness Brain

**The shared commit-memory for projects managed by [Harness Stack](https://github.com/cloudbloqavi/harness-stack).**

Harness Brain is a plain-Markdown, git-backed memory of *what changed and why*
across every project in your ecosystem. The `commit-brain-agent` writes a
compact dated summary here on each commit; the `cross-repo-discovery-agent`
reads it at session start to surface a recent-change digest across a project and
its dependencies.

It is intentionally simple: human-readable Markdown, one folder per project, one
file per day. No database, no service — just a repo you can grep, diff, and read.

```
   project repo (on_commit)                          new session start
          |                                                  |
          v                                                  v
  +-------------------+                            +---------------------------+
  | commit-brain-     |   append summary           | cross-repo-discovery-     |
  |   agent           |--------------+             |   agent                   |
  +-------------------+              |             +-------------+-------------+
                                     v                           ^
                        +---------------------------------+      | read 7/30-day
                        |          harness-brain          |      | digest across
                        |  projects/<project>/            |------+ project + deps
                        |    <YYYY-MMM-DD>.har.compact.md  |
                        +---------------------------------+
                          (located via HARNESS_BRAIN_PATH)
```

## Layout

```
harness-brain/
├── README.md
├── projects/
│   └── <project-name>/
│       ├── README.md                    ← per-project index (auto-bootstrapped)
│       └── <YYYY-MMM-DD>.har.compact.md  ← dated change log (appended per commit)
└── _compact/
    └── YYYY-MMM-DD.har.compact.md       ← the format each daily entry follows
```

- **One folder per project**, named after the repo.
- **One file per day**, `YYYY-MMM-DD.har.compact.md` (e.g. `2026-Jun-06.har.compact.md`).
  Multiple commits on the same day append to the same file.
- Folders + per-project README are bootstrapped on the project's first commit.

### One repo per folder — even for multi-repo products

A folder maps to a **repository**, not a product. A product built from several
repositories therefore gets **one folder per repo**, and the relationship
between them is declared in each folder's `README.md` (a `Part of:` product line
plus `Related repos:` / `Depends on:` links). The `cross-repo-discovery-agent`
uses those declarations to decide which folders to read **together** at session
start. Two scenarios, both worked through in [`projects/`](projects/):

| Scenario | Example folders | Relationship |
| --- | --- | --- |
| **Related** — one product, many repos | `example-related--ledger-api`, `example-related--ledger-web` | Same product (Ledger); `web` depends on `api`. Read together as one cross-repo digest. |
| **Unrelated** — independent repos | `example-unrelated--weather-cli`, `example-unrelated--markdown-linter` | No relationship; each digested in isolation. |

The `example-related--` / `example-unrelated--` prefixes are illustrative labels
so the scenario is obvious from the folder name. In a real brain the folder is
simply the repo name — relatedness lives in the README declarations, not the
folder name.

## Entry format

Each commit appends a section (≤200 words) capturing: what changed, why, the
files touched, and any to-dos or flags. See
[`_compact/YYYY-MMM-DD.har.compact.md`](_compact/YYYY-MMM-DD.har.compact.md) for
the canonical shape and the worked examples under
[`projects/`](projects/) (related vs unrelated repos — see Layout above).

## How it is written

The `commit-brain-agent` (defined in `harness-stack/.subagents/`) runs on
`on_commit`, reads the diff + commit message, and appends a summary here. It is
designed to **never block a commit** — on any error it logs to stderr and exits
0.

Point the agent at this repo with the `HARNESS_BRAIN_PATH` environment variable:

```bash
export HARNESS_BRAIN_PATH=/path/to/harness-brain
```

## How it is read

The `cross-repo-discovery-agent` runs at session start, collects the last
7/30 days of `.har.compact.md` files for the current project and its declared
dependencies, and produces a per-project digest. If a dependency has no folder
here, it flags it and asks whether to add it to the brain.
