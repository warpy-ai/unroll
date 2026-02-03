# Contributing to Unroll

Thank you for your interest in contributing to Unroll! This document provides guidelines for contributing to the project.

## Getting Started

### Prerequisites

- Oite compiler (`oitec`) installed
- Git for version control
- A text editor or IDE with Oite support

### Setting Up the Development Environment

1. Clone the repository:
   ```bash
   git clone https://github.com/warpy-ai/unroll.git
   cd unroll
   ```

2. Build the project:
   ```bash
   oitec build src/main.ot -o unroll
   ```

3. Run tests:
   ```bash
   ./unroll test
   ```

## Development Workflow

### Branching Strategy

- `main` - Stable release branch
- `develop` - Development branch
- `feature/*` - Feature branches
- `fix/*` - Bug fix branches

### Creating a Feature Branch

```bash
git checkout develop
git pull origin develop
git checkout -b feature/my-feature
```

### Commit Messages

Follow the conventional commits specification:

```
type(scope): description

[optional body]

[optional footer]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

Examples:
```
feat(cli): add --watch flag to build command
fix(resolver): handle circular dependencies correctly
docs(readme): update installation instructions
```

## Code Style

### General Guidelines

1. **Indentation**: Use 4 spaces (no tabs)
2. **Line Length**: Maximum 100 characters
3. **Naming**:
   - Functions: `camelCase`
   - Types/Interfaces: `PascalCase`
   - Constants: `UPPER_SNAKE_CASE`
   - Variables: `camelCase`

### Example Code Style

```typescript
// Good
interface BuildOptions {
    release: boolean;
    target: string;
    verbose: boolean;
}

function runBuild(options: BuildOptions): number {
    if (options.verbose) {
        console.log("Building in " + options.target + " mode...");
    }
    
    let result = compileSources(options);
    return result.success ? 0 : 1;
}

// Bad
interface build_options {
  Release:boolean;
    target : string;
verbose: boolean }

function run_build(opts: build_options) : number{
if(opts.verbose){console.log("Building...");}
let result=compileSources(opts);return result.success?0:1;}
```

### Comments

- Use `//` for single-line comments
- Use `/* */` for multi-line comments
- Add JSDoc-style comments for public functions:

```typescript
/// Compile the project sources
/// @param options - Build configuration options
/// @returns Compilation result with success status and errors
function compileSources(options: BuildOptions): CompileResult {
    // Implementation
}
```

## Testing

### Writing Tests

Create test files in the `tests/` directory with the naming convention `*_test.ot`:

```typescript
// tests/resolver_test.ot

import { resolveDependencies } from "../src/resolver/mod.ot";

function testBasicResolution(): boolean {
    let manifest = createTestManifest();
    let result = resolveDependencies(manifest);
    
    assert(result.error == "", "Should resolve without error");
    assert(result.packages.length > 0, "Should have packages");
    
    return true;
}

function testCircularDependency(): boolean {
    let manifest = createCircularManifest();
    let result = resolveDependencies(manifest);
    
    assert(result.error != "", "Should detect circular dependency");
    
    return true;
}
```

### Running Tests

```bash
# Run all tests
./unroll test

# Run specific tests
./unroll test --filter "resolver*"

# Run with verbose output
./unroll test --verbose
```

## Pull Request Process

1. **Create a Pull Request**:
   - Target the `develop` branch
   - Use a clear, descriptive title
   - Fill out the PR template

2. **PR Description Should Include**:
   - Summary of changes
   - Motivation and context
   - How to test the changes
   - Screenshots (if applicable)

3. **Review Process**:
   - At least one approval required
   - All CI checks must pass
   - No merge conflicts

4. **After Approval**:
   - Squash and merge to `develop`
   - Delete the feature branch

## Reporting Issues

### Bug Reports

Include:
- Clear, descriptive title
- Steps to reproduce
- Expected behavior
- Actual behavior
- Environment (OS, Oite version, Unroll version)
- Error messages and logs

### Feature Requests

Include:
- Clear, descriptive title
- Problem you're trying to solve
- Proposed solution
- Alternative solutions considered
- Additional context

## Project Structure

```
unroll/
├── src/
│   ├── main.ot             # Entry point
│   ├── cli/                # CLI commands
│   ├── config/             # Configuration handling
│   ├── resolver/           # Dependency resolution
│   ├── registry/           # Registry client
│   ├── build/              # Build system
│   └── lsp/                # Language server (future)
├── tests/                  # Test files
├── docs/                   # Documentation
├── examples/               # Example projects
└── README.md
```

## Getting Help

- **Documentation**: Check the `docs/` directory
- **Issues**: Search existing issues before creating new ones
- **Discussions**: Use GitHub Discussions for questions

## Code of Conduct

### Our Pledge

We are committed to providing a friendly, safe, and welcoming environment for all contributors.

### Expected Behavior

- Be respectful and inclusive
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards others

### Unacceptable Behavior

- Harassment or discrimination
- Trolling or insulting comments
- Personal or political attacks
- Publishing others' private information

## License

By contributing to Unroll, you agree that your contributions will be licensed under the Apache License 2.0.

## Recognition

Contributors will be recognized in:
- The AUTHORS file
- Release notes for significant contributions
- The project README (for major contributors)

Thank you for contributing to Unroll!
