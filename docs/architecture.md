# Unroll Architecture

This document describes the internal architecture of Unroll, the package manager and build system for Script.

## Overview

Unroll is designed as a modular system with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                         CLI Layer                           │
│  src/cli/*.tscl - Command parsing and orchestration         │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      Core Services                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Config    │  │  Resolver   │  │      Registry       │  │
│  │  manifest   │  │  semver     │  │      client         │  │
│  │  lockfile   │  │  deps graph │  │      auth           │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      Build Pipeline                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Compiler   │  │   Linker    │  │   Test Runner       │  │
│  │  interface  │  │   LTO       │  │   discovery         │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                    Script Compiler                          │
│  External: scriptc binary for actual compilation            │
└─────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
src/
├── main.tscl           # Entry point, command dispatch
├── cli/                # CLI command implementations
│   ├── args.tscl       # Argument parsing
│   ├── new.tscl        # unroll new
│   ├── init.tscl       # unroll init
│   ├── deps.tscl       # add/remove/update
│   ├── build.tscl      # build/run/watch/clean
│   ├── test.tscl       # test runner
│   ├── fmt.tscl        # code formatter
│   ├── lint.tscl       # linter
│   ├── check.tscl      # type checker
│   ├── doc.tscl        # documentation generator
│   └── registry.tscl   # search/info/publish/login
├── config/             # Configuration handling
│   ├── manifest.tscl   # unroll.toml parsing
│   ├── lockfile.tscl   # unroll.lock parsing
│   └── scaffold.tscl   # project scaffolding
├── resolver/           # Dependency resolution
│   └── mod.tscl        # Version resolution algorithm
├── registry/           # Package registry client
│   ├── client.tscl     # HTTP client for registry
│   └── config.tscl     # Registry configuration
├── build/              # Build system
│   ├── compiler.tscl   # Compiler interface
│   ├── linker.tscl     # Linking and LTO
│   └── test_discovery.tscl
└── lsp/                # Language Server Protocol
    └── (future)
```

## Key Components

### 1. CLI Layer (`src/cli/`)

The CLI layer handles user interaction:

- **Argument Parsing**: Converts command-line arguments into typed options
- **Command Dispatch**: Routes to appropriate handler based on command
- **Output Formatting**: Consistent error messages and progress output

### 2. Configuration (`src/config/`)

Handles project configuration files:

- **Manifest (`unroll.toml`)**: Project metadata, dependencies, build settings
- **Lockfile (`unroll.lock`)**: Exact resolved versions for reproducibility
- **Scaffold**: Templates for new projects

### 3. Dependency Resolver (`src/resolver/`)

Resolves dependency versions:

```
Input: unroll.toml dependencies
       ┌─────────────────────┐
       │ "@rolls/http" = "0.1"│
       │ "@rolls/json" = "*"  │
       └──────────┬──────────┘
                  │
                  ▼
       ┌─────────────────────┐
       │   Fetch Versions    │
       │   from Registry     │
       └──────────┬──────────┘
                  │
                  ▼
       ┌─────────────────────┐
       │  Resolve Conflicts  │
       │  (semver matching)  │
       └──────────┬──────────┘
                  │
                  ▼
Output: unroll.lock
       ┌─────────────────────┐
       │ @rolls/http = 0.1.5 │
       │ @rolls/json = 0.2.1 │
       │ @rolls/async = 0.1.2│
       └─────────────────────┘
```

### 4. Registry Client (`src/registry/`)

Communicates with the package registry:

- **Search**: Find packages by name/keyword
- **Fetch**: Download package metadata and tarballs
- **Publish**: Upload new package versions
- **Auth**: Handle authentication tokens

### 5. Build System (`src/build/`)

Orchestrates the compilation process:

```
Source Files (.tscl)
        │
        ▼
┌───────────────────┐
│   Compile Phase   │──► Object Files (.o)
│   (via scriptc)   │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│    Link Phase     │──► Binary / Library
│   (clang/lld)     │
└───────────────────┘
```

## Data Flow

### Build Command Flow

```
unroll build --release
        │
        ▼
┌───────────────────┐
│ Parse Arguments   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Load unroll.toml  │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Load unroll.lock  │──► If missing, run resolver
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Compile Sources   │──► Parallel compilation
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Compile Deps      │──► Use cached if available
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Link Binary       │──► Apply LTO if release
└─────────┬─────────┘
          │
          ▼
     ./target/release/app
```

### Dependency Add Flow

```
unroll add @rolls/http
        │
        ▼
┌───────────────────┐
│ Parse Package Spec│──► "@rolls/http@*"
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Load unroll.toml  │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Add to deps list  │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Resolve All Deps  │──► Fetch from registry
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Write unroll.toml │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Write unroll.lock │
└─────────────────────┘
```

## Caching Strategy

Unroll uses a multi-level cache:

```
~/.unroll/
├── cache/
│   ├── registry/           # Package metadata cache
│   │   └── @rolls/
│   │       └── http/
│   │           └── versions.json
│   ├── packages/           # Downloaded package tarballs
│   │   └── @rolls/
│   │       └── http-0.1.5.tar.gz
│   └── builds/             # Compiled dependency objects
│       └── @rolls/
│           └── http-0.1.5/
│               └── release/
│                   └── http.o
├── config.toml             # Global configuration
└── credentials             # Registry tokens
```

## Error Handling

All operations return result types with explicit errors:

```typescript
interface Result<T> {
    success: boolean;
    value: T | null;
    error: string;
}
```

Errors are propagated up the call stack with context added at each level.

## Future: LSP Server

The LSP server will provide IDE integration:

- Completion (types, functions, imports)
- Hover (type information)
- Go to Definition
- Find References
- Diagnostics
- Code Actions
- Formatting

## Performance Considerations

1. **Parallel Compilation**: Use multiple CPU cores
2. **Incremental Builds**: Only recompile changed files
3. **Dependency Caching**: Pre-compiled dependency objects
4. **Registry Caching**: Avoid redundant network requests
5. **Memory Mapping**: Use mmap for large files
