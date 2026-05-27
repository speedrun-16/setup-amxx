# setup-amxx

GitHub Action that downloads the AMXXPawn compiler, caches it, and adds it to `PATH`.

On **Linux and macOS** the stdlib include directory is injected automatically into every `amxxpc` call via a thin wrapper, so you only need to pass `-i` flags for project-specific includes. On **Windows** the directory is on `PATH` and `AMXXPAWN_INCLUDE` points to the includes.

## Usage

```yaml
- uses: actions/checkout@v4

- uses: speedrun-16/setup-amxx@main
  with:
    release-tag: 'latest'

- name: Compile plugin
  run: |
    amxxpc src/plugin.sma -i"src/include" -o out/plugin.amxx
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `release-tag` | no | `latest` | Specific AMXX GitHub release tag or `latest` to resolve automatically |
| `plugin` | no | | Path to the main `.sma` file to parse the plugin version from |
| `version-define` | no | | `#define` name to read the version from (example: `PLUGIN_VERSION`). If omitted, looks for a `version = "x.x.x"` assignment |

## Outputs

| Output | Description |
|--------|-------------|
| `release-tag` | Resolved GitHub release tag that was installed |
| `scripting-path` | Absolute path to the scripting directory (contains `amxxpc`) |
| `include-path` | Absolute path to the stdlib include directory |
| `plugin-version` | Plugin version parsed from `plugin` (empty if `plugin` was not set) |

## Environment variables

| Variable | Description |
|----------|-------------|
| `AMXXPAWN_SCRIPTING` | Same as `scripting-path` output |
| `AMXXPAWN_INCLUDE` | Same as `include-path` output |
| `LD_LIBRARY_PATH` | (Linux only) Prepended with the scripting directory so the compiler can find its shared libraries |

## Parsing the plugin version from a .sma file

```yaml
- uses: speedrun-16/setup-amxx@main
  id: setup
  with:
    release-tag: 'latest'
    plugin: src/plugin.sma
    version-define: PLUGIN_VERSION   # reads: #define PLUGIN_VERSION "1.2.3"

- run: echo "Plugin version is ${{ steps.setup.outputs.plugin-version }}"
```

If `version-define` is omitted the action looks for a `version = "x.x.x"` assignment in the file instead.

## Pinning to a specific release

```yaml
- uses: speedrun-16/setup-amxx@main
  with:
    release-tag: '1.10.0.5476'
```

Pinning is recommended for reproducible builds. With `latest`, the resolved GitHub release tag is available in `release-tag`.

Downloads are resolved from the official AMX Mod X GitHub releases, for example:

```text
https://github.com/alliedmodders/amxmodx/releases/download/1.10.0.5476/amxmodx-1.10.0-git5476-base-linux.tar.gz
```

## Caching

The compiler is cached between runs using `actions/cache`. \
Cache key includes the release tag and OS so different platforms never share a cache entry. \
Cache hit skips the download entirely.
