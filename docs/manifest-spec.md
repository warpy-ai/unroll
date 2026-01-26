# Unroll Manifest Specification (unroll.toml)

This document describes the `unroll.toml` manifest file format used to configure Script projects.

## Overview

Every Unroll project contains an `unroll.toml` file at its root. This file defines:
- Package metadata
- Dependencies
- Build configuration
- Feature flags

## Sections

### [package]

Required package metadata.

```toml
[package]
name = "my-app"                     # Required: Package name
version = "0.1.0"                   # Required: SemVer version
edition = "2025"                    # Script language edition
license = "MIT"                     # SPDX license identifier
description = "Short description"  # Package description
repository = "https://..."          # Source repository URL
homepage = "https://..."            # Project homepage
documentation = "https://..."       # Documentation URL
keywords = ["web", "server"]        # Search keywords (max 5)
categories = ["web-programming"]    # Category slugs
authors = ["Name <email@example.com>"]
readme = "README.md"                # Readme file path
include = ["src/**/*"]              # Files to include in package
exclude = ["tests/**/*"]            # Files to exclude from package
```

#### Package Types

```toml
[package]
# Binary (default)
name = "my-app"

# Library
[lib]
name = "my_lib"
path = "src/lib.tscl"

# Both binary and library
[[bin]]
name = "my-app"
path = "src/main.tscl"

[lib]
name = "my_lib"
path = "src/lib.tscl"
```

### [dependencies]

Runtime dependencies.

```toml
[dependencies]
# Simple version
"@rolls/http" = "0.1"

# Version with features
"@rolls/tls" = { version = "0.1", features = ["native-roots"] }

# Optional dependency
"@rolls/json" = { version = "0.1", optional = true }

# Git dependency
"my-lib" = { git = "https://github.com/user/my-lib", branch = "main" }

# Path dependency (local development)
"local-lib" = { path = "../local-lib" }
```

#### Version Requirements

| Syntax | Meaning |
|--------|---------|
| `"0.1"` | `>=0.1.0, <0.2.0` (default) |
| `"=0.1.5"` | Exactly 0.1.5 |
| `">=0.1.5"` | Greater than or equal |
| `"<0.2.0"` | Less than |
| `">=0.1, <0.3"` | Range |
| `"0.1.*"` | Any patch version |
| `"*"` | Any version |

### [dev-dependencies]

Development-only dependencies (tests, benchmarks).

```toml
[dev-dependencies]
"@rolls/test" = "0.1"
"@rolls/bench" = "0.1"
```

### [build-dependencies]

Build script dependencies.

```toml
[build-dependencies]
"@rolls/codegen" = "0.1"
```

### [features]

Feature flags for conditional compilation.

```toml
[features]
# Default features enabled
default = ["json"]

# Feature definitions
json = ["@rolls/json"]
full = ["json", "tls", "@rolls/websocket"]
tls = ["@rolls/tls"]

# Feature that enables optional dependency
http2 = ["dep:@rolls/h2"]
```

Usage in code:
```javascript
// Feature-gated code
#[cfg(feature = "json")]
import { JSON } from "@rolls/json";
```

### [build]

Build configuration.

```toml
[build]
target = "native"           # native | wasm32 | wasm32-wasi
optimization = "release"    # debug | release | dist
lto = true                  # Enable link-time optimization
codegen-units = 1           # Fewer units = better optimization
```

### [profile.*]

Build profiles for different scenarios.

```toml
[profile.dev]
opt-level = 0               # No optimization
debug = true                # Include debug info
incremental = true          # Incremental compilation
overflow-checks = true      # Integer overflow checks

[profile.release]
opt-level = 3               # Maximum optimization
debug = false               # No debug info
lto = "thin"                # ThinLTO
strip = "symbols"           # Strip symbols

[profile.dist]
inherits = "release"        # Inherit from release
lto = "fat"                 # Full LTO
codegen-units = 1           # Single codegen unit
strip = "all"               # Strip all metadata
```

#### Optimization Levels

| Level | Description |
|-------|-------------|
| `0` | No optimization (fastest compile) |
| `1` | Basic optimization |
| `2` | Standard optimization |
| `3` | Maximum optimization |
| `"s"` | Optimize for size |
| `"z"` | Aggressively optimize for size |

#### LTO Options

| Value | Description |
|-------|-------------|
| `false` | No LTO |
| `true` / `"thin"` | ThinLTO (parallel, faster) |
| `"fat"` | Full LTO (slower, smaller binary) |

### [workspace]

Workspace configuration for monorepos.

```toml
[workspace]
members = [
    "packages/*",
    "crates/my-lib",
]
exclude = [
    "packages/experimental",
]

# Shared dependencies for all workspace members
[workspace.dependencies]
"@rolls/http" = "0.1"
"@rolls/json" = "0.1"
```

### [target.*]

Target-specific configuration.

```toml
# Linux-specific dependencies
[target.'cfg(target_os = "linux")'.dependencies]
"@rolls/io-uring" = "0.1"

# macOS-specific dependencies
[target.'cfg(target_os = "macos")'.dependencies]
"@rolls/grand-central" = "0.1"

# WebAssembly-specific
[target.wasm32-unknown-unknown.dependencies]
"@rolls/wasm-bindgen" = "0.1"
```

### [badges]

Status badges for documentation.

```toml
[badges]
maintenance = { status = "actively-developed" }

[badges.github]
repository = "user/repo"
workflow = "CI"
```

## Environment Variables

Reference environment variables in manifest:

```toml
[package]
version = { env = "PACKAGE_VERSION", default = "0.0.0" }

[registries.private]
token = { env = "PRIVATE_REGISTRY_TOKEN" }
```

## Complete Example

```toml
[package]
name = "my-web-server"
version = "1.0.0"
edition = "2025"
license = "MIT"
description = "A fast web server built with Script"
repository = "https://github.com/example/my-web-server"
keywords = ["web", "server", "http"]
authors = ["Developer <dev@example.com>"]

[dependencies]
"@rolls/http" = { version = "0.1", features = ["http2"] }
"@rolls/tls" = { version = "0.1", optional = true }
"@rolls/json" = "0.1"
"@rolls/async" = "0.1"

[dev-dependencies]
"@rolls/test" = "0.1"

[features]
default = ["json"]
json = ["@rolls/json"]
https = ["@rolls/tls", "@rolls/http/tls"]
full = ["json", "https"]

[profile.release]
opt-level = 3
lto = "thin"

[profile.dist]
inherits = "release"
lto = "fat"
strip = "all"

[[bin]]
name = "server"
path = "src/main.tscl"

[lib]
name = "my_web_server"
path = "src/lib.tscl"
```
