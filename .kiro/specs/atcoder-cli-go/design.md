# Design Document

## Overview

The AtCoder CLI Go tool will be a command-line application built using Go's standard library and popular CLI frameworks. The application will follow a modular architecture with clear separation of concerns between authentication, web scraping, file management, and command execution. The tool will interact with AtCoder's web interface through HTTP requests and HTML parsing since AtCoder doesn't provide a public API.

## Architecture

The application will follow a layered architecture:

```
┌─────────────────────────────────────┐
│           CLI Commands              │
├─────────────────────────────────────┤
│         Service Layer               │
├─────────────────────────────────────┤
│      Repository Layer               │
├─────────────────────────────────────┤
│    External Dependencies            │
└─────────────────────────────────────┘
```

### Core Components:
- **CLI Layer**: Command definitions and argument parsing using cobra
- **Service Layer**: Business logic for each major feature
- **Repository Layer**: Data access and external API interactions
- **Configuration**: Settings management and persistence
- **Authentication**: Session management and credential handling

## Components and Interfaces

### 1. CLI Commands
```go
type Command interface {
    Execute(args []string) error
    Validate(args []string) error
}
```

Commands to implement:
- `login` - Authenticate with AtCoder
- `download` - Download contest problems
- `test` - Run local tests
- `submit` - Submit solution
- `config` - Manage configuration
- `info` - Display contest/problem information

### 2. AtCoder Client
```go
type AtCoderClient interface {
    Login(username, password string) error
    GetContest(contestID string) (*Contest, error)
    GetProblem(contestID, problemID string) (*Problem, error)
    SubmitSolution(contestID, problemID string, code []byte, language string) (*Submission, error)
    GetSubmissionStatus(submissionID string) (*SubmissionStatus, error)
}
```

### 3. File Manager
```go
type FileManager interface {
    CreateProblemDirectory(contestID, problemID string) error
    SaveProblemStatement(contestID, problemID string, content []byte) error
    SaveTestCases(contestID, problemID string, testCases []TestCase) error
    GenerateTemplate(contestID, problemID, language string) error
    ReadSolutionFile(contestID, problemID string) ([]byte, error)
}
```

### 4. Test Runner
```go
type TestRunner interface {
    CompileSolution(filePath string, language string) error
    RunTests(contestID, problemID string) (*TestResults, error)
    RunSingleTest(executablePath string, input []byte) (*TestResult, error)
}
```

### 5. Configuration Manager
```go
type ConfigManager interface {
    Load() (*Config, error)
    Save(config *Config) error
    GetDefaultLanguage() string
    GetWorkspaceDir() string
    GetTemplateDir() string
}
```

## Data Models

### Contest
```go
type Contest struct {
    ID          string
    Name        string
    StartTime   time.Time
    Duration    time.Duration
    Problems    []Problem
}
```

### Problem
```go
type Problem struct {
    ID          string
    ContestID   string
    Title       string
    TimeLimit   time.Duration
    MemoryLimit int64
    Statement   string
    TestCases   []TestCase
}
```

### TestCase
```go
type TestCase struct {
    Input    string
    Expected string
}
```

### Configuration
```go
type Config struct {
    DefaultLanguage string            `json:"default_language"`
    WorkspaceDir    string            `json:"workspace_dir"`
    TemplateDir     string            `json:"template_dir"`
    Templates       map[string]string `json:"templates"`
}
```

## Error Handling

### Error Types
- **AuthenticationError**: Login failures, session expiry
- **NetworkError**: Connection issues, timeouts
- **ParsingError**: HTML parsing failures
- **FileSystemError**: File I/O operations
- **CompilationError**: Code compilation failures
- **ValidationError**: Input validation failures

### Error Handling Strategy
- Use Go's standard error handling with custom error types
- Provide user-friendly error messages
- Log detailed errors for debugging
- Implement retry logic for network operations
- Graceful degradation when possible

## Testing Strategy

### Unit Testing
- Test each service component in isolation
- Mock external dependencies (HTTP client, file system)
- Test error conditions and edge cases
- Achieve >80% code coverage

### Integration Testing
- Test CLI commands end-to-end
- Test against AtCoder's actual website (with rate limiting)
- Test file system operations
- Test configuration persistence

### Test Structure
```
tests/
├── unit/
│   ├── services/
│   ├── repositories/
│   └── utils/
├── integration/
│   ├── cli/
│   └── atcoder/
└── fixtures/
    ├── html/
    └── testdata/
```

### Testing Tools
- Go's built-in testing framework
- testify for assertions and mocking
- httptest for HTTP client testing
- Golden files for HTML parsing tests

## Implementation Details

### Authentication Flow
1. User provides credentials via CLI
2. Perform login POST request to AtCoder
3. Extract and store session cookies
4. Validate authentication by accessing protected page
5. Persist session data securely

### Problem Download Flow
1. Fetch contest page HTML
2. Parse problem list from HTML
3. For each problem:
   - Fetch problem page HTML
   - Parse problem statement and test cases
   - Create directory structure
   - Save files (statement.md, test cases, template)

### Testing Flow
1. Detect programming language from file extension
2. Compile solution if necessary
3. Run solution against each test case
4. Compare output with expected results
5. Display results with colored output

### Submission Flow
1. Read solution file
2. Detect programming language
3. Authenticate if necessary
4. Submit via POST request
5. Poll submission status
6. Display final result

## Security Considerations

- Store authentication cookies securely (encrypted)
- Validate all user inputs
- Sanitize file paths to prevent directory traversal
- Rate limit requests to AtCoder
- Handle sensitive data (passwords) securely in memory