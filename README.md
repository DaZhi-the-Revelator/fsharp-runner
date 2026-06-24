# fsharp-runner

GitHub Actions runner for [Keen](https://codeberg.org/DaZhi-the-Revelator/keen) binary releases.

Triggered manually by supplying a Keen version tag. Builds Keen and its .NET sidecar for all supported platforms, then creates a release on Codeberg using the matching changelog entry as the release body.


## How It Works

Each build job:

1. Clones the Keen repository at the supplied tag
2. Compiles the Rust binary for the target platform
3. Publishes the .NET sidecar as a self-contained single-file executable
4. Stages both the `keen` binary and the `sidecar/` subdirectory into an archive

The release job downloads all build artifacts, extracts the matching entry from `Changelog.md`, and creates a Codeberg release with the archives attached.


## Archive Contents

Each platform archive contains:

```txt
keen/
├── keen              # (keen.exe on Windows)
├── README.md
├── LICENSE
└── sidecar/
    └── keen-sidecar  # (keen-sidecar.exe on Windows)
```

The `sidecar/` directory must remain adjacent to the `keen` binary at runtime.


## Platforms

| Archive | Target |
|---|---|
| `keen-linux-x86_64.tar.gz` | Linux x86_64 (musl static) |
| `keen-linux-aarch64.tar.gz` | Linux aarch64 (musl static) |
| `keen-windows-x86_64.zip` | Windows x86_64 |
| `keen-macos-arm64.tar.gz` | macOS Apple Silicon |
| `keen-macos-x86_64.tar.gz` | macOS Intel |


## Usage

Go to **Actions → Build and Release → Run workflow** and enter the Keen tag to build (e.g. `v0.3.0`).

The tag must exist on the Keen Codeberg repository. The `Changelog.md` entry for that tag is used as the release body. If no matching entry is found, the release is created with an empty body.


## Secrets

| Secret | Purpose |
|---|---|
| `CODEBERG_TOKEN` | Gitea API token with `write:release` scope on the Keen repository |


## Relationship to Keen's Forgejo CI

Keen's `.forgejo/workflows/trigger_github.yml` triggers on `v*` tag pushes and dispatches this workflow on GitHub. Keen's `.forgejo/workflows/release.yml` is a disabled placeholder; all building and releasing happens here, on GitHub, exclusively.
