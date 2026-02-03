# Unroll LSP Server Design

This document describes the planned design for the Unroll Language Server Protocol (LSP) implementation.

## Overview

The LSP server provides IDE integration for Oite, including:

- Real-time diagnostics (type errors, lint warnings)
- Auto-completion
- Hover information
- Go to definition
- Find references
- Rename symbol
- Code formatting
- Code actions

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Editor/IDE                          │
│  (VS Code, Neovim, Helix, Zed, etc.)                       │
└─────────────────────────────┬───────────────────────────────┘
                              │ JSON-RPC over stdio
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      LSP Server                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Transport  │  │   Router    │  │    Capabilities     │  │
│  │  (stdio)    │  │  (dispatch) │  │    (negotiate)      │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      Analysis Engine                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Parser    │  │ Type Check  │  │     Borrow Check    │  │
│  │   (AST)     │  │ (inference) │  │     (ownership)     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      Index/Database                         │
│  Symbol table, type information, cross-references           │
└─────────────────────────────────────────────────────────────┘
```

## Supported LSP Methods

### Lifecycle

| Method | Description |
|--------|-------------|
| `initialize` | Negotiate capabilities |
| `initialized` | Initialization complete |
| `shutdown` | Prepare to exit |
| `exit` | Terminate server |

### Text Document Sync

| Method | Description |
|--------|-------------|
| `textDocument/didOpen` | Document opened |
| `textDocument/didChange` | Document modified |
| `textDocument/didSave` | Document saved |
| `textDocument/didClose` | Document closed |

### Language Features

| Method | Description |
|--------|-------------|
| `textDocument/completion` | Auto-complete |
| `textDocument/hover` | Type/doc info |
| `textDocument/definition` | Go to definition |
| `textDocument/references` | Find all references |
| `textDocument/rename` | Rename symbol |
| `textDocument/formatting` | Format document |
| `textDocument/codeAction` | Quick fixes |
| `textDocument/publishDiagnostics` | Error/warning reporting |

### Workspace

| Method | Description |
|--------|-------------|
| `workspace/symbol` | Find symbol in workspace |
| `workspace/configuration` | Get settings |

## File Structure

```
src/lsp/
├── mod.ot              # Main LSP server
├── transport.ot        # JSON-RPC transport
├── messages.ot         # LSP message types
├── capabilities.ot     # Server capabilities
├── handlers/
│   ├── lifecycle.ot    # init/shutdown
│   ├── document.ot     # didOpen/didChange/etc
│   ├── completion.ot   # Auto-complete
│   ├── hover.ot        # Hover info
│   ├── definition.ot   # Go to definition
│   ├── references.ot   # Find references
│   ├── rename.ot       # Rename symbol
│   ├── formatting.ot   # Code formatting
│   └── diagnostics.ot  # Error reporting
├── analysis/
│   ├── parser.ot       # Incremental parser
│   ├── typecheck.ot    # Type checking
│   ├── symbols.ot      # Symbol table
│   └── index.ot        # Cross-reference index
└── util/
    ├── position.ot     # Position conversion
    └── uri.ot          # URI handling
```

## Completion

### Trigger Characters

- `.` - Member access
- `"` - Import path
- `@` - Decorator
- `/` - Path completion

### Completion Kinds

| Kind | Example |
|------|---------|
| Function | `function greet(name: string)` |
| Variable | `let count = 0` |
| Class | `class HttpServer` |
| Interface | `interface Request` |
| Property | `.status` |
| Method | `.listen()` |
| Keyword | `if`, `while`, `import` |
| Snippet | `for (let i = 0; ...)` |

### Example Completion Response

```json
{
  "isIncomplete": false,
  "items": [
    {
      "label": "listen",
      "kind": 2,
      "detail": "(port: number) => void",
      "documentation": "Start the server listening on the given port"
    },
    {
      "label": "close",
      "kind": 2,
      "detail": "() => void",
      "documentation": "Stop the server"
    }
  ]
}
```

## Diagnostics

### Severity Levels

| Level | Description |
|-------|-------------|
| Error | Type errors, syntax errors |
| Warning | Unused variables, deprecated APIs |
| Information | Hints and suggestions |
| Hint | Style recommendations |

### Example Diagnostic

```json
{
  "uri": "file:///project/src/main.ot",
  "diagnostics": [
    {
      "range": {
        "start": { "line": 10, "character": 4 },
        "end": { "line": 10, "character": 15 }
      },
      "severity": 1,
      "code": "E0001",
      "source": "unroll",
      "message": "Cannot assign type 'string' to type 'number'"
    }
  ]
}
```

## Incremental Updates

The LSP server uses incremental parsing for performance:

```
Document Change
      │
      ▼
┌───────────────────┐
│ Identify Changed  │
│ Range             │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Re-parse Affected │
│ Syntax Nodes      │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Update Symbol     │
│ Table             │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Re-typecheck      │
│ Affected Scopes   │
└─────────┬─────────┘
          │
          ▼
   Publish Diagnostics
```

## Configuration

### VS Code Settings

```json
{
  "unroll.trace.server": "verbose",
  "unroll.inlayHints.enabled": true,
  "unroll.inlayHints.parameterNames": true,
  "unroll.inlayHints.typeHints": true,
  "unroll.diagnostics.enable": true,
  "unroll.formatting.enable": true,
  "unroll.formatting.tabSize": 4
}
```

### Neovim Configuration

```lua
require('lspconfig').unroll.setup {
  cmd = { 'unroll', 'lsp' },
  filetypes = { 'ot', 'oite' },
  root_dir = function(fname)
    return lspconfig.util.root_pattern('unroll.toml')(fname)
  end,
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Initial indexing | <5s for 100k LOC |
| Completion response | <50ms |
| Hover response | <20ms |
| Go to definition | <30ms |
| Incremental update | <100ms |

## Future Enhancements

1. **Inlay Hints**: Show inferred types inline
2. **Semantic Tokens**: Rich syntax highlighting
3. **Call Hierarchy**: Show callers/callees
4. **Type Hierarchy**: Show class inheritance
5. **Folding Ranges**: Code folding support
6. **Selection Range**: Smart selection expansion
