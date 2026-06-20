## Local CLI Runner

This checkout includes a root-level `bench` command for running the official Bench Packs without the Electron app.

The Bench Packs are frozen local snapshots under `benchpacks/`. Their upstream repo, tag, commit, git tree, and source hash are pinned in `benchpacks.lock.json`; the nested pack `.git` directories are intentionally removed. Normal `./bench sync` restores or verifies the lock only. Updating from the remote registry requires the explicit `./bench update-lock --force` flow and should be followed by a fresh audit.

### Quick Start

```bash
cd ~/src/benchlocal

# Safe default: run non-Docker packs only.
./bench llamacpp http://127.0.0.1:9292/v1 light --model-match qwen3.5-397b

# Run every pack, including Docker verifier/agent packs.
./bench llamacpp http://127.0.0.1:9292/v1 all --model-match qwen3.5-397b --allow-docker

# Run one pack.
./bench llamacpp http://127.0.0.1:9292/v1 instruct --model qwen3.5-397b-a17b-nvfp4
```

If no output file is supplied, results are written automatically under:

```text
bench-results/<model-id>/<model>__mode-...__packs-...__params-<hash>.json
```

Existing files are never overwritten; repeated runs get `-2`, `-3`, etc. Add `--label <name>` to make result files easier to scan.

### Command Shape

```bash
./bench <mode> <url> <bench_pack_types> [output_file|auto] [options]
```

Common modes:

- `llamacpp`
- `ollama`
- `lmstudio`
- `mlx`
- `openai`
- `openrouter`

Pack selectors:

- `all`: every known pack
- `light` or `simple`: non-Docker packs only
- `toolcall`, `bugfind`, `dataextract`, `instruct`, `reasonmath`, `structoutput`, `hermes`, `cli`
- comma-separated selectors, for example `instruct,reason,tool`

Useful options:

- `--model <id>`: exact model id
- `--model-match <text>`: select one model from `<url>/models`
- `--scenarios <ids>`: comma-separated scenario ids
- `--temperature <n>`: default `0`
- `--timeout <seconds>`: default `600`
- `--allow-docker`: opt into Docker verifier/agent packs
- `--output-dir <dir>`: auto-output directory, default `bench-results`
- `--label <text>`: prefix auto-output filenames
- `--no-build`: skip pack build step

### Safety And Audit

```bash
./bench audit bench-results/audit.json
```

The audit checks the frozen source hash, npm lifecycle hooks, `npm audit`, secret-looking patterns, dynamic code patterns, and Dockerfile/runtime risk. Docker packs are skipped unless `--allow-docker` is passed.

Docker-backed packs currently are:

- `bugfind-15`
- `structoutput-15`
- `hermesagent-20`
- `cli-40`

For local model endpoints bound to `127.0.0.1`, the runner starts a temporary Docker-accessible relay during Docker pack runs. It closes the relay when the run finishes.

### Frozen Pack Maintenance

```bash
# Verify or restore exactly what benchpacks.lock.json pins.
./bench sync

# Re-freeze current pack snapshots and strip nested .git dirs.
./bench freeze

# Deliberate remote update flow. Review audit output afterward.
./bench update-lock --force
./bench audit bench-results/audit.json
```

Prefer `light` for quick, low-risk local checks. Use `--allow-docker` when you want the full verifier/agent picture.

<p align="center">
  <img src="./docs/assets/benchlocal-logo.svg" alt="BenchLocal logo" width="104" />
</p>

<h1 align="center">BenchLocal</h1>

<p align="center">
  Test LLMs on real tasks. Compare models side-by-side.
</p>

<p align="center">
  <a href="https://benchlocal.com">Website</a>
  ·
  <a href="https://github.com/stevibe/BenchLocal/releases/latest">Download</a>
  ·
  <a href="./docs/assets/benchlocal-demo.mp4">Watch demo</a>
  ·
  <a href="./BENCH_PACK_AUTHORING.md">Build a Bench Pack</a>
</p>

<p align="center">
  <a href="./docs/assets/benchlocal-demo.mp4">
    <img src="./screenshot.png" alt="BenchLocal desktop app preview" />
  </a>
</p>

BenchLocal is a local-first desktop app for running, comparing, and managing installable LLM Bench Packs against local or remote models.

Official Bench Packs today:

- [ToolCall-15](https://github.com/stevibe/ToolCall-15)
- [BugFind-15](https://github.com/stevibe/BugFind-15)
- [DataExtract-15](https://github.com/stevibe/DataExtract-15)
- [InstructFollow-15](https://github.com/stevibe/InstructFollow-15)
- [ReasonMath-15](https://github.com/stevibe/ReasonMath-15)
- [StructOutput-15](https://github.com/stevibe/StructOutput-15)
- [CLI-40](https://github.com/stevibe/CLI-40)
- [HermesAgent-20](https://github.com/stevibe/HermesAgent-20)

BenchLocal owns the shared desktop runtime:

- provider configuration
- model registry
- Bench Pack install and update flow
- per-tab sampling overrides
- run execution and result history
- verifier lifecycle management
- persisted desktop UI state

Each Bench Pack owns its benchmark behavior:

- scenario definitions
- benchmark-specific prompts
- scoring logic
- verifier contracts where required
- benchmark-specific traces and summaries

## Repo layout

- `app/`
  Electron app shell, desktop UI, main process, preload, renderer
- `packages/benchlocal-core`
  shared protocol, config, workspace, and theme types
- `packages/benchlocal-sdk`
  authoring helpers for Bench Pack repos
- `packages/benchpack-host`
  host-side install, inspection, verifier, and run orchestration logic
- `themes/`
  built-in desktop themes
- `scripts/`
  local macOS release helpers
- `docs/`
  packaging and release docs

## Developer references

- [ARCHITECTURE.md](./ARCHITECTURE.md)
- [BENCH_PACK_AUTHORING.md](./BENCH_PACK_AUTHORING.md)
- [BENCH_PROTOCOL_V1.md](./BENCH_PROTOCOL_V1.md)
- [CONFIG_SCHEMA_V1.md](./CONFIG_SCHEMA_V1.md)
- [BENCHLOCAL_REGISTRY_V1.md](./BENCHLOCAL_REGISTRY_V1.md)
- [docs/macos-release.md](./docs/macos-release.md)
- [docs/windows-release.md](./docs/windows-release.md)
- [docs/linux-release.md](./docs/linux-release.md)

## Build commands

- `npm run build`
  compile the app and workspace packages for development
- `npm run pack`
  compile and package the production desktop app, including DMG and ZIP artifacts
- `npm run build:dir`
  compile and produce an unpacked local app bundle
- `npm run build:win`
  compile and package unsigned Windows NSIS and ZIP artifacts
- `npm run build:linux`
  compile and package Linux AppImage and tar.gz artifacts
- `npm run release:all`
  build the signed macOS release plus Windows and Linux desktop artifacts in one command

## License

MIT
