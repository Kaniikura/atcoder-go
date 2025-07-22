# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go-based CLI tool for AtCoder competitive programming platform. It's designed as a modern replacement for the unmaintained atcoder-cli project, providing contest download, local testing, and solution submission capabilities through web scraping.

## Essential Commands

### Development
```bash
# Build the CLI
go build -o atcoder-cli ./cmd/atcoder-cli

# Run tests
go test ./...                    # All tests
go test ./internal/client -v     # Specific package with verbose output
go test -cover ./...            # With coverage report
go test -race ./...             # With race condition detection

# Install locally
go install ./cmd/atcoder-cli
```

### CLI Usage (once implemented)
```bash
atcoder-cli login                           # Authenticate with AtCoder
atcoder-cli download <contest-id>           # Download contest problems
atcoder-cli test                           # Run tests for current problem
atcoder-cli test -problem <problem-id>     # Run tests for specific problem
atcoder-cli submit                         # Submit solution
atcoder-cli submit -problem <problem-id>   # Submit specific problem
```

## Architecture Overview

The project follows a layered architecture:

```
CLI Layer (cmd/) → Service Layer (internal/) → External APIs (AtCoder website)
```

### Key Components

1. **CLI Commands** (`cmd/atcoder-cli/`)
   - Uses Cobra framework for command structure
   - Each command delegates to service layer

2. **HTTP Client** (`internal/client/`)
   - Handles authentication and session management
   - Implements web scraping with proper error handling
   - Stores encrypted cookies for persistent login

3. **Contest Management** (`internal/contest/`)
   - Models for Contest, Problem, TestCase
   - Handles problem parsing and test case extraction

4. **File Manager** (`internal/filemanager/`)
   - Creates contest/problem directory structure
   - Manages solution files and test cases
   - Handles template instantiation

5. **Test Runner** (`internal/runner/`)
   - Executes solutions against test cases
   - Supports multiple languages (Go, Python, C++, Java)
   - Provides detailed test results with diffs

6. **Submission Handler** (`internal/submit/`)
   - Handles solution submission to AtCoder
   - Monitors submission status
   - Reports results

## Development Guidelines

### Testing Strategy
- Write unit tests for all new functionality
- Use interfaces for external dependencies (HTTP, filesystem)
- Mock external services in tests using testify/mock
- Maintain >80% code coverage
- Use golden files for HTML parsing tests in `tests/fixtures/`

### Error Handling
- Create specific error types for different failure modes
- Always wrap errors with context using `fmt.Errorf`
- Provide user-friendly error messages in CLI layer

### Security Considerations
- Store cookies encrypted using `golang.org/x/crypto`
- Validate all user inputs before use
- Implement rate limiting for AtCoder requests
- Never log sensitive information

### Code Patterns
- Use dependency injection through interfaces
- Keep business logic in service layer, not in CLI commands
- Use context.Context for cancellation and timeouts
- Follow standard Go project layout

## Implementation Status

The project is currently in initial development phase. Key tasks from the implementation plan:

1. Project setup and structure - **Not started**
2. Configuration management - **Not started**
3. Authentication module - **Not started**
4. Problem downloader - **Not started**
5. Test runner - **Not started**
6. Solution submitter - **Not started**
7. CLI commands - **Not started**
8. Error handling - **Not started**
9. Documentation - **Not started**
10. Testing & CI/CD - **Not started**

## Key Files to Reference

- `.kiro/specs/atcoder-cli-go/requirements.md` - User stories and acceptance criteria
- `.kiro/specs/atcoder-cli-go/design.md` - Technical architecture details
- `.kiro/specs/atcoder-cli-go/tasks.md` - Implementation task breakdown

## Common Development Tasks

### Adding a New Command
1. Create command file in `cmd/atcoder-cli/commands/`
2. Implement service logic in appropriate `internal/` package
3. Add unit tests for both command and service
4. Update CLI help documentation

### Updating HTML Parsing
1. Add sample HTML to `tests/fixtures/`
2. Update parser in `internal/client/parser.go`
3. Add/update golden test files
4. Test against actual AtCoder pages

### Adding Language Support
1. Add language config to `internal/runner/languages.go`
2. Implement compilation/execution logic
3. Add test cases for the new language
4. Update documentation