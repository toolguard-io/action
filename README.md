# ToolGuard Action

**MCP contract checks in your pull requests** — classified diffs (breaking / compatible / suspicious) against a committed baseline, plus spec readiness for the MCP **2025-11-25** and **2026-07-28** versions.

> 🚧 **In development** (M5 of the [ToolGuard](https://github.com/toolguard-io/toolguard) MVP). The Action wraps the [`mcp-toolguard`](https://pypi.org/project/mcp-toolguard/) CLI: it snapshots your server, diffs it against `.toolguard/baseline.json`, validates against the spec matrix and posts a single, updatable comment on the PR.

## Planned usage

```yaml
- uses: toolguard-io/action@v1
  with:
    target: "python -m my_mcp_server"   # or an http(s) URL
    # policy: breaking                  # audit | breaking (default) | strict
    # spec-versions: "2025-11-25,2026-07-28"
```

Until it ships, you can already gate any CI with the CLI:

```bash
uvx mcp-toolguard ci "python -m my_mcp_server" --markdown >> "$GITHUB_STEP_SUMMARY"
```

## License

MIT — see [LICENSE](LICENSE).
