# Local fork notes (Onebrownsound/benchlocal `local` branch)

This fork tracks upstream `stevibe/benchlocal` and carries our local CLI runner
(`bench`), the vendored-pack lockfile, and one pack patch — without ever
hand-editing vendored pack files inside this repo.

## Remotes / branch
- `upstream` = stevibe/benchlocal · `origin` = Onebrownsound/benchlocal · work on `local`.

## The dataextract patch lives in its OWN fork (not vendored here)
- Fork: `Onebrownsound/DataExtract-15`, branch `patches`, tag `v1.0.0-local`
  (= upstream v1.0.0 + `stripCodeFence()` so the scorer strips ```json markdown
  fences before JSON.parse; upstream scored correct-but-fenced output as 0).
- `benchpacks.lock.json` pins `dataextract-15` at that fork tag/commit, so
  `./bench sync` materializes the patched pack with a MATCHING hash — no `--force`,
  no silent clobber. `benchpacks/` stays gitignored (reproduced from the lock).

## Update from upstream
    git fetch upstream && git merge upstream/main   # brings new harness versions
    npm install                                     # if root deps changed (e.g. undici)
    # rebuild app only if running Electron; ./bench run rebuilds packs on demand

## Update the dataextract patch when its upstream moves
    cd <DataExtract-15 clone>
    git fetch upstream --tags && git rebase upstream/<new-tag> patches
    git tag <new-tag>-local && git push -f origin patches --tags
    # back here: edit benchpacks.lock.json dataextract source.{tag,commit,tree} -> new fork ref,
    # then: ./bench sync --force && ./bench freeze --keep-git   # repins the hash

## TODO
- Upstream the fence fix as a PR to stevibe/DataExtract-15; when merged, the fork
  patch becomes redundant and the lockfile can point back at upstream.
