# ToolGuard Action

**MCP contract checks in your pull requests** — classified diffs (breaking / compatible / suspicious) against a committed baseline, plus spec readiness for the MCP **2025-11-25** and **2026-07-28** versions. Wraps the [`mcp-toolguard`](https://pypi.org/project/mcp-toolguard/) CLI from [ToolGuard](https://github.com/toolguard-io/toolguard).

On every PR the action snapshots your server, diffs it against `.toolguard/baseline.json`, validates it against the spec matrix, gates the check by policy, and keeps **a single, updatable comment** on the PR (no comment spam on new pushes).

## Quickstart

1. Create and commit a baseline once, from your repo root:

   ```bash
   uvx mcp-toolguard baseline "python -m my_mcp_server"
   git add .toolguard/baseline.json && git commit -m "Add ToolGuard baseline"
   ```

2. Add the workflow:

   ```yaml
   name: toolguard
   on: [pull_request]

   jobs:
     contract:
       runs-on: ubuntu-latest
       permissions:
         contents: read
         pull-requests: write   # for the PR comment
       steps:
         - uses: actions/checkout@v4
         - uses: toolguard-io/action@v1
           with:
             target: "python -m my_mcp_server"   # or an http(s) URL
   ```

When the contract changes intentionally, accept it with `uvx mcp-toolguard baseline <target> --update` and commit — the acceptance is visible in review.

## Inputs

| Input | Default | Description |
|---|---|---|
| `target` | *(required)* | http(s) URL (streamable HTTP) or stdio command, e.g. `"python -m my_mcp_server"` |
| `policy` | `breaking` | `audit` never fails · `breaking` fails on breaking changes or spec errors · `strict` fails on any change or any finding above info |
| `baseline` | `.toolguard/baseline.json` | Path to the committed baseline |
| `spec-versions` | CLI matrix | MCP spec versions to validate, comma-separated |
| `timeout` | `30` | Connection timeout in seconds |
| `comment` | `true` | Post/update the single PR comment (pull_request events only) |
| `comment-id` | `toolguard` | Use distinct ids to keep separate comments when checking several servers in one PR |
| `github-token` | `${{ github.token }}` | Token for the PR comment (needs `pull-requests: write`) |
| `version` | latest | Pin a specific `mcp-toolguard` release, e.g. `"0.1.0"` |

## Outputs

| Output | Description |
|---|---|
| `passed` | `"true"` when the policy gate passed |
| `exit-code` | `0` ok · `1` policy failure · `2` operational error (connection, missing baseline…) |
| `report` | Path to the Markdown report (also appended to the step summary) |

## Checking several servers

Run the action once per server with distinct `comment-id`s:

```yaml
- uses: toolguard-io/action@v1
  with:
    target: https://mcp.example.com/mcp
    comment-id: example-mcp
- uses: toolguard-io/action@v1
  with:
    target: "python -m internal_server"
    comment-id: internal
```

## Notes

- The report always lands in the **step summary**, even on `push` events or with `comment: false`.
- A missing or stale baseline is an **operational error** (exit 2), not a policy verdict — the comment tells you how to fix it.
- Not on GitHub? The CLI gates any CI: `uvx mcp-toolguard ci <target> --policy breaking`.

## License

MIT — see [LICENSE](LICENSE).
