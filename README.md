<div align="center">
  <h1>Unroll</h1>
  <p>Package Manager, Build System, and Developer Tooling for Script</p>

  <br/>

  <img src="https://img.shields.io/badge/status-in--development-yellow.svg" alt="Status"/>
  <img src="https://img.shields.io/badge/license-Apache%202.0-blue.svg" alt="License"/>
</div>

---

## Overview

**Unroll** is the package manager, build system, and developer tooling ecosystem for the [Script](https://github.com/aspect-build/script) language. It manages Rolls dependencies, handles compilation, and provides developer experience features like LSP support.

```
┌─────────────────────────────────────────┐
│            User App Code                │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│   Rolls (official system libs)          │
│   @rolls/http, @rolls/tls, @rolls/db    │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│   Unroll (runtime + tooling)            │  ← THIS REPO
│   pkg manager, lockfiles, bundler, LSP  │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│   Script (language core)                │
│   compiler, type system, ABI, bootstrap │
└─────────────────────────────────────────┘
```

## Features

- **Dependency Management** - Add, update, and remove Rolls with lockfile support
- **Build System** - Compile Script projects to native binaries with LTO
- **Cross-Compilation** - Target multiple platforms from a single machine
- **Language Server** - Full LSP support for IDE integration
- **Code Formatting** - Built-in formatter (`unroll fmt`)
- **Linting** - Static analysis and best practices (`unroll lint`)
- **Testing** - Integrated test runner with coverage support
- **Documentation** - Auto-generate docs from code comments

## Quick Start

### Installation

```bash
# Build from source (requires Rust)
cargo install --path .

# Or download prebuilt binary
curl -fsSL https://script-lang.org/install-unroll.sh | sh
```

### Create a New Project

```bash
# Create a new application
unroll new my-app
cd my-app

# Or create a library
unroll new my-lib --lib
```

### Project Structure

```
my-app/
├── unroll.toml         # Project manifest
├── unroll.lock         # Lockfile (auto-generated)
├── src/
│   ├── main.tscl       # Entry point
│   └── lib.tscl        # Library root (for --lib projects)
├── tests/
│   └── integration_test.tscl
├── benches/
│   └── perf_bench.tscl
├── examples/
│   └── basic_usage.tscl
└── target/
    ├── debug/
    │   └── my-app      # Debug binary
    └── release/
        └── my-app      # Release binary
```

## CLI Commands

### Project Management

```bash
unroll new <name>           # Create new project
unroll new <name> --lib     # Create library project
unroll init                 # Initialize in existing directory
```

### Dependency Management

```bash
unroll add @rolls/http              # Add dependency
unroll add @rolls/tls --optional    # Add optional dependency
unroll add --dev @rolls/test        # Add dev dependency
unroll remove @rolls/http           # Remove dependency
unroll update                       # Update all dependencies
unroll update @rolls/http           # Update specific dependency
```

### Build & Run

```bash
unroll build                        # Debug build
unroll build --release              # Release build with ThinLTO
unroll build --dist                 # Distribution build with Full LTO
unroll build --target wasm32        # Cross-compile to WebAssembly

unroll run                          # Build and run
unroll run --release                # Run release build

unroll watch                        # Watch mode (rebuild on changes)
unroll clean                        # Clean build artifacts
```

### Testing

```bash
unroll test                         # Run all tests
unroll test --filter "http*"        # Run filtered tests
unroll test --coverage              # Run with coverage report
```

### Developer Tools

```bash
unroll fmt                          # Format code
unroll fmt --check                  # Check formatting (CI)

unroll lint                         # Run linter
unroll lint --fix                   # Auto-fix issues

unroll check                        # Type check only

unroll doc                          # Generate documentation
unroll doc --open                   # Generate and open in browser
```

### Registry

```bash
unroll search <query>               # Search for packages
unroll info @rolls/http             # View package info
unroll info @rolls/http --versions  # List all versions

unroll login                        # Authenticate with registry
unroll publish                      # Publish a Roll
unroll deprecate <version>          # Deprecate a version
```

## Project Manifest (unroll.toml)

```toml
[package]
name = "my-app"
version = "0.1.0"
edition = "2025"
license = "MIT"
description = "My awesome app"
repository = "https://github.com/example/my-app"
keywords = ["web", "server"]

[dependencies]
"@rolls/http" = "0.1"
"@rolls/json" = "0.1"

[dev-dependencies]
"@rolls/test" = "0.1"

[features]
default = ["json"]
json = ["@rolls/json"]
full = ["json", "@rolls/tls"]

[build]
target = "native"           # native | wasm32
optimization = "release"    # debug | release
lto = true                  # Link-time optimization

[profile.release]
opt-level = 3
lto = "thin"

[profile.dev]
opt-level = 0
debug = true

[profile.dist]
opt-level = 3
lto = "fat"
strip = true
```

## Lockfile (unroll.lock)

Unroll automatically generates and maintains a lockfile for reproducible builds:

```toml
# This file is auto-generated by Unroll. Do not edit.
# unroll.lock format version 1

[[package]]
name = "@rolls/http"
version = "0.1.5"
source = "registry+https://rolls.script-lang.org"
checksum = "sha256:abcd1234..."
dependencies = [
    "@rolls/async 0.1.2",
    "@rolls/tls 0.1.0 (optional)",
]

[[package]]
name = "@rolls/async"
version = "0.1.2"
source = "registry+https://rolls.script-lang.org"
checksum = "sha256:efgh5678..."
```

## Language Server Protocol (LSP)

Unroll includes a full-featured LSP server for IDE integration.

### Features

- Auto-completion for types, functions, and imports
- Hover information with types and documentation
- Go to definition / Find references
- Rename symbol refactoring
- Real-time diagnostics
- Code actions and quick fixes
- Integrated formatting

### VS Code Setup

```json
// .vscode/settings.json
{
  "unroll.path": "/usr/local/bin/unroll",
  "unroll.trace.server": "verbose",
  "unroll.inlayHints.enabled": true,
  "unroll.checkOnSave": true
}
```

## Cross-Compilation Targets

| Target                      | Description                 |
| --------------------------- | --------------------------- |
| `x86_64-unknown-linux-gnu`  | Linux x64 (glibc)           |
| `x86_64-unknown-linux-musl` | Linux x64 (static)          |
| `aarch64-unknown-linux-gnu` | Linux ARM64                 |
| `x86_64-apple-darwin`       | macOS x64                   |
| `aarch64-apple-darwin`      | macOS ARM64 (Apple Silicon) |
| `x86_64-pc-windows-msvc`    | Windows x64                 |
| `wasm32-unknown-unknown`    | WebAssembly                 |
| `wasm32-wasi`               | WASM + WASI                 |

```bash
# Install target toolchain
unroll target add aarch64-unknown-linux-gnu

# Cross-compile
unroll build --target aarch64-unknown-linux-gnu
```

## Registry Configuration

### Public Registry

The default registry is at `https://rolls.script-lang.org`.

### Private Registry

Configure private registries for enterprise use:

```toml
# ~/.unroll/config.toml
[registries]
default = "https://rolls.script-lang.org"
company = "https://rolls.company.internal"

[registries.company]
token = "env:COMPANY_REGISTRY_TOKEN"
```

## Performance Targets

| Metric                | Target       |
| --------------------- | ------------ |
| Cold start            | <100ms       |
| Incremental build     | <500ms       |
| LSP response          | <50ms        |
| Dependency resolution | <1s (cached) |

## Roadmap

- [x] Project structure design
- [ ] Core CLI framework
- [ ] Manifest parsing (unroll.toml)
- [ ] Lockfile generation
- [ ] Dependency resolver
- [ ] Registry client
- [ ] Build system integration
- [ ] LSP server
- [ ] Code formatter
- [ ] Linter
- [ ] Test runner
- [ ] Documentation generator
- [ ] Workspace support
- [ ] Plugin system

## Related Projects

- [Script](https://github.com/aspect-build/script) - The Script language compiler
- [Rolls](https://github.com/aspect-build/rolls) - Official system libraries

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

Unroll is distributed under the terms of the Apache License (Version 2.0).

See [LICENSE](LICENSE) for details.
