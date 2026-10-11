<div align="center">

<pre align="center">
██╗   ██╗███████╗████████╗████████╗ ██████╗ 
██║   ██║██╔════╝╚══██╔══╝╚══██╔══╝██╔═══██╗
██║   ██║█████╗     ██║      ██║   ██║   ██║
╚██╗ ██╔╝██╔══╝     ██║      ██║   ██║   ██║
 ╚████╔╝ ███████╗   ██║      ██║   ╚██████╔╝
  ╚═══╝  ╚══════╝   ╚═╝      ╚═╝    ╚═════╝ 
</pre>

<p align="center">
  <strong>Vetto</strong> is an unprivileged kernel sandbox and security runtime for AI coding agents.
</p>

[![CI](https://img.shields.io/github/actions/workflow/status/shleder/vetto/ci.yml?branch=main&label=CI&style=flat-square)](https://github.com/shleder/vetto/actions)
[![Release](https://img.shields.io/github/v/release/shleder/vetto?label=release&color=blue&style=flat-square)](https://github.com/shleder/vetto/releases)
[![npm version](https://img.shields.io/badge/npm-v0.6.3-CB3837?logo=npm&logoColor=white&style=flat-square)](https://www.npmjs.com/package/@shledery/vetto)
[![crates.io](https://img.shields.io/badge/crates.io-v0.6.3-orange?logo=rust&logoColor=white&style=flat-square)](https://crates.io/crates/vetto)
[![Platforms](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey?style=flat-square)](https://github.com/shleder/vetto)
[![License: Apache 2.0](https://img.shields.io/badge/license-Apache--2.0-green?style=flat-square)](LICENSE)

<p align="center">
  <img src="assets/readme/demo.gif" alt="Vetto Terminal Demo" width="880">
</p>

</div>

Vetto enforces filesystem boundaries, network egress allowlists, and process tree containment directly between `fork()` and `execve()` with sub-millisecond cold start latency. It requires zero root privileges and runs without a background daemon.

---

## Quickstart

### Installation

Install via standalone shell script:

```bash
curl -fsSL https://raw.githubusercontent.com/shleder/vetto/main/install.sh | sh
```

Or via your preferred package manager:

```bash
# npm (cross-platform global binary)
npm install -g @shledery/vetto

# Homebrew (macOS & Linux)
brew install shleder/tap/vetto

# Cargo (crates.io)
cargo install vetto --locked
```

<details>
<summary>Manual binary download from GitHub Releases</summary>

Download pre-built archives from [GitHub Releases](https://github.com/shleder/vetto/releases/latest):
- Linux x86_64: `vetto-x86_64-unknown-linux-gnu.tar.gz`
- Linux aarch64: `vetto-aarch64-unknown-linux-gnu.tar.gz`
- macOS Apple Silicon: `vetto-aarch64-apple-darwin.tar.gz`
- macOS Intel: `vetto-x86_64-apple-darwin.tar.gz`
- Windows x86_64: `vetto-x86_64-pc-windows-msvc.zip`

Each archive includes SHA-256 checksums and minisign signatures (`.minisig`).
</details>

---

## Usage

### 1. Transparent Agent Wrapping

Wrap all detected AI coding agents (`claude`, `codex`, `cursor`, `opencode`, `aider`) with a single command:

```bash
vetto enable --all
```

Run your agent as usual. Vetto transparently intercepts execution via lightweight shims in `~/.vetto/shims`:

```bash
claude                # runs sandboxed under kernel LSM policies
codex                 # host credentials masked, network scoped to OpenAI API
```

To unwrap agents or restore direct execution:

```bash
vetto disable claude  # disable sandbox for claude
vetto disable --all   # remove all shims
```

### 2. Direct Sandbox Execution

Run arbitrary commands, test suites, or agent scripts inside a one-off sandbox:

```bash
vetto run -- claude
vetto -- npm test
```

### 3. Workspace Snapshot & Rollback

Vetto captures a copy-on-write snapshot before your agent runs:

```bash
vetto diff            # review modified, added, or removed files
vetto undo            # instantly revert all changes to pre-session state
```

### 4. Diagnostics & Verification

Inspect host kernel isolation features and verify security boundaries:

```bash
vetto doctor          # inspect Landlock ABI, namespaces, and cgroups v2
vetto verify          # run boundary verification suite against active policy
vetto audit --latest  # inspect cryptographic Merkle audit log of the last session
```

---

## Isolation Model

| Platform | Filesystem | Network | Process Containment | Startup Overhead |
| :--- | :--- | :--- | :--- | :--- |
| **Linux (Tier 1)** | Landlock LSM (ABI 1–6) + CoW tmpfs | `CLONE_NEWNET` namespaces + SNI relay | cgroups v2 `cgroup.kill` + `pidfd` pinning | `< 4ms` |
| **macOS (Tier 2)** | Seatbelt (`libsandbox.1.dylib`) | Loopback proxy + IP egress deny | Child process group supervision | `< 8ms` |
| **Windows (Tier 3)** | LPAC AppContainer tokens | Restricted network SIDs | Job Objects (`KILL_ON_JOB_CLOSE`) | `< 15ms` |

---

## Documentation

- [Platform Backends & Isolation Specs](docs/platform-backends.md)
- [Agent Compatibility Registry (32+ agents)](docs/agents.md)
- [SWE-bench vs Docker Runtime Benchmark](docs/benchmarks/swe-bench.md)
- [GitHub Actions CI/CD Integration](docs/ci-cd.md)
- [Threat Model & Boundary Architecture](docs/threat-model.md)
- [Exit Codes & Failure Modes](docs/exit-codes.md)
- [Security Policy](SECURITY.md)

---

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
