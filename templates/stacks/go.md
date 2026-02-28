# Go Stack

## Tech Stack

- **Language:** Go
- **Build:** `go build`
- **Module system:** Go modules (`go.mod`)

## Test Commands

```bash
# All tests
go test ./...

# Specific package
go test ./pkg/mypackage/...

# Specific test
go test ./pkg/mypackage/ -run TestMyFunction

# With race detection
go test -race ./...

# With coverage
go test -cover ./...
```

## Lint Commands

```bash
# Format (auto-fix)
gofmt -w .

# Vet (static analysis)
go vet ./...

# golangci-lint (comprehensive)
golangci-lint run ./...
```

## Build Commands

```bash
# Build all binaries
go build ./cmd/...

# Build specific binary
go build -o bin/myservice ./cmd/myservice
```

## Recommended Configuration

### warnings-as-errors

Go's compiler already treats most issues as errors. Add `golangci-lint` for comprehensive checks:

```yaml
# .golangci.yml
linters:
  enable:
    - errcheck       # unchecked errors
    - govet          # suspicious constructs
    - staticcheck    # advanced static analysis
    - unused         # unused code
    - gosimple       # simplifications
    - ineffassign    # ineffective assignments
linters-settings:
  errcheck:
    check-type-assertions: true
    check-blank: true
```

### Test framework

Go's built-in `testing` package. Use `testify` for assertion helpers if desired, but the standard library is sufficient.

### BDD framework

Godog (`github.com/cucumber/godog`). Feature files in `features/`, step definitions alongside test files.

### Pre-commit hook

```bash
#!/bin/sh
# .git/hooks/pre-commit
set -e
gofmt -l . | grep -q . && echo "Run gofmt" && exit 1
go vet ./...
go test ./...
```
