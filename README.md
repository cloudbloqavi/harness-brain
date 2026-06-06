# Harness Brain

**The shared commit-memory for projects managed by [Harness Stack](https://github.com/cloudbloqavi/harness-stack).**

Harness Brain is a plain-Markdown, git-backed memory of *what changed and why*
across every project in your ecosystem. The `commit-brain-agent` writes a
compact dated summary here on each commit; the `cross-repo-discovery-agent`
reads it at session start to surface a recent-change digest across a project and
its dependencies.

It is intentionally simple: human-readable Markdown, one folder per project, one
file per day. No database, no service — just a repo you can grep, diff, and read.

## Layout

```
harness-brain/
├── README.md
├── projects/
│   └── <project-name>/
│       ├── README.md                ← per-project index (auto-bootstrapped)
│       └── <YYYY-MM-DD>.harness.md   ← dated change log (appended per commit)
└── _template/
    └── DAILY.harness.md             ← the format each daily entry follows
```

- **One folder per project**, named after the repo.
- **One file per day**, `YYYY-MM-DD.harness.md`. Multiple commits on the same
  day append to the same file.
- Folders + per-project README are bootstrapped on the project's first commit.

## Entry format

Each commit appends a section (≤200 words) capturing: what changed, why, the
files touched, and any to-dos or flags. See
[`_template/DAILY.harness.md`](_template/DAILY.harness.md) for the canonical
shape and [`projects/example-project`](projects/example-project) for a worked
example.

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
7/30 days of `.harness.md` files for the current project and its declared
dependencies, and produces a per-project digest. If a dependency has no folder
here, it flags it and asks whether to add it to the brain.
