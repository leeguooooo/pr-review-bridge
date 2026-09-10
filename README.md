# PR Review Bridge

A Gogs review service that spends model calls only when there is work worth reviewing.

Authors declare `none`, `spark`, or `deep` in the PR body. The service verifies webhook signatures, waits for a stable revision, reviews a pinned checkout, and publishes findings with private author notifications. Equivalent source reviews share a disk cache. Closed, merged and replaced revisions cancel work.

```text
[review:none] Documentation only; runtime behavior is unchanged
[review:spark] Local validation change
[review:deep] Transaction and retry behavior changes
```

## What works

- Codex SDK backend: author-selected Spark low or GPT-6 low, with bounded context requests and no automatic complexity routing.
- Exact-source result and stage caches, 60-second settling, bounded execution, stale-result rejection.
- Gogs API robot identity, signed webhooks, durable SQLite queues and editable machine-readable status.
- Lark bot DMs: new actionable findings only; no group fallback or zero-findings spam.
- Two-person acknowledgement checker; service outages can use the manual path without erasing existing defects.

## Start

Python **3.11+**, Git, an authenticated Codex CLI, and a Gogs robot with repository read/comment access are required. Lark DMs additionally require an authenticated `lark-cli` bot.

```sh
python3 -m venv .venv
.venv/bin/pip install -r pr_review/requirements.txt
.venv/bin/python -m unittest discover -s tests
.venv/bin/python pr_review/service.py /private/config.json
```

See [integration and deployment](pr_review/README.md), [cross-session integration](docs/cross-session.md), and [technical walkthrough](docs/technical-sharing.md).

## Boundaries

This is an experimental integration. A healthy HTTP endpoint is not proof a model review succeeded. A browser-backed `chatgpt-use` adapter is implemented and offline-tested; production activation and live end-to-end verification remain deployment-specific. Model quotas remain those of the selected provider.

Gogs is the currently implemented Git host. The checker is **not** a native server-enforced merge lock. Automated merging requires a separate explicitly configured merge identity and host integration; it is not enabled by this public release. A skipped review is not an AI approval.

Runtime credentials, identity mappings, logs, reviewed source and cache artifacts belong outside this repository. This publication is a clean source export, with no production Git history or runtime state.

MIT license. Contributions and reproducible issues welcome.
