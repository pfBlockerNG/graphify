## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- After modifying code files in this session, run `graphify update .` to keep the graph current (AST-only, no API cost)

## Downstream integration builds

In the `pfBlockerNG/graphify` fork, `origin/integration` is the canonical source for
builds with all downstream patches applied, based on `upstream/v8`
(`Graphify-Labs/graphify`). Keep upstream PR branches separate; cherry-pick their
reviewed changes onto `integration`. The dedicated local worktree is
`/root/git/.graphify_worktrees/integration`; the primary checkout is `/root/git/graphify`.

PR heads live where their pull request was opened. `feat/omp-native-integration`
(#3506) is headed by `andrebrait/graphify`; push it to both `andrebrait` and `origin`
so the fork holds every branch. `andrebrait/graphify` carries nothing else.

Refresh the branch by fetching `origin` and `upstream`, fast-forwarding to
`origin/integration`, creating a uniquely named `backup/integration-*` branch, and
rebasing onto `upstream/v8`. Preserve every downstream behavior when resolving
conflicts; drop a patch only after confirming upstream provides its complete behavior
(the interpreter-gated `leiden` extra was dropped this way at 0.9.60). Compare the
replayed stack with the backup and rerun verification. Push additions normally; after
a rebase, use `--force-with-lease=refs/heads/integration:<previous-remote-sha>` with
the exact remote SHA captured before rebasing.

Verify from the integration worktree: `uv lock` must be a no-op, then
`uv run --frozen --extra leiden pytest tests/test_language_overrides.py
tests/test_language_override_imports.py tests/test_language_override_lifecycle.py
tests/test_backend_extras.py tests/test_omp_install.py` and
`bun test tests/omp.test.ts` (needs `@oh-my-pi/pi-coding-agent` resolvable, e.g. a
temporary `node_modules` symlink to an OMP extension package's dependencies).

Install the tool from the full SHA of `origin/integration`, never from a feature
branch or a dirty checkout:

```sh
uv tool install --force --reinstall \
  'graphifyy[leiden] @ git+https://github.com/pfBlockerNG/graphify@<full-sha>'
```

`~/.local/share/uv/tools/graphifyy/uv-receipt.toml` records the pin. The current
stack: project language overrides (upstream #3075 plus fork fixes) and the native OMP
hook bridge (#3506).
