---
title: Go Best Practices
inclusion: fileMatch
fileMatchPattern: '**/*.go,**/go.mod,**/go.sum'
---

# Go Best Practices

## Go File Types and Organization
- **`.go`** - Go source code files
- **`go.mod`** - Go module definition file (dependencies and module info)
- **`go.sum`** - Go module checksums for dependency verification
- **`main.go`** - Entry point for executable programs (package main)
- **`*_test.go`** - Test files (automatically excluded from builds)
- **`doc.go`** - Package documentation files
- Use `internal/` directories for private packages
- Use `cmd/` directory for multiple executables in one module
- Use `pkg/` directory for library code intended for external use

```go
// Project structure example
myproject/
├── go.mod
├── go.sum
├── main.go              // Main executable
├── doc.go              // Package documentation
├── cmd/
│   ├── server/
│   │   └── main.go     // Server executable
│   └── client/
│       └── main.go     // Client executable
├── internal/           // Private packages
│   ├── config/
│   │   └── config.go
│   └── database/
│       └── db.go
├── pkg/               // Public library code
│   └── api/
│       └── client.go
└── testdata/          // Test fixtures
    └── sample.json
```

## Code Style and Formatting
- Use `gofmt` to format code consistently
- Use `goimports` to manage imports automatically
- Follow Go naming conventions: PascalCase for exported, camelCase for unexported
- Use meaningful variable and function names
- Keep functions small and focused on single responsibilities

```go
// Good: Clear, descriptive names
func CalculateMonthlyPayment(principal, rate float64, months int) float64 {
    return principal * rate / (1 - math.Pow(1+rate, -float64(months)))
}

// Bad: Unclear abbreviations
func calc(p, r float64, m int) float64 {
    return p * r / (1 - math.Pow(1+r, -float64(m)))
}
```

## Error Handling
- Always handle errors explicitly, never ignore them
- Use custom error types for domain-specific errors
- Wrap errors with context using `fmt.Errorf` with `%w` verb
- Return errors as the last return value

```go
// Good: Explicit error handling with context
func ReadConfig(filename string) (*Config, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("failed to read config file %s: %w", filename, err)
    }
    
    var config Config
    if err := json.Unmarshal(data, &config); err != nil {
        return nil, fmt.Errorf("failed to parse config: %w", err)
    }
    
    return &config, nil
}
```

## Package Organization
- Use short, descriptive package names
- Avoid package names like `util`, `common`, or `helper`
- Group related functionality in packages
- Keep `main` package minimal, move logic to other packages
- Use internal packages for implementation details

```go
// Good package structure
// pkg/
//   auth/
//     auth.go
//     token.go
//   storage/
//     database.go
//     cache.go
//   internal/
//     config/
//       config.go
```

## Concurrency
- Use channels to communicate between goroutines
- Use `sync.WaitGroup` for waiting on multiple goroutines
- Use `context.Context` for cancellation and timeouts
- Avoid shared mutable state, prefer message passing
- Use `sync.Once` for one-time initialization

```go
// Good: Using channels and context
func ProcessItems(ctx context.Context, items []Item) error {
    results := make(chan Result, len(items))
    errors := make(chan error, len(items))
    
    for _, item := range items {
        go func(item Item) {
            select {
            case <-ctx.Done():
                errors <- ctx.Err()
                return
            default:
                result, err := processItem(item)
                if err != nil {
                    errors <- err
                    return
                }
                results <- result
            }
        }(item)
    }
    
    // Collect results...
    return nil
}
```

## Testing
- Write table-driven tests for multiple test cases
- Use descriptive test names that explain the scenario
- Test both success and failure cases
- Use `testing.T.Helper()` for test helper functions
- Use `go test -race` to detect race conditions

```go
func TestCalculateMonthlyPayment(t *testing.T) {
    tests := []struct {
        name      string
        principal float64
        rate      float64
        months    int
        want      float64
    }{
        {
            name:      "standard mortgage",
            principal: 200000,
            rate:      0.05/12,
            months:    360,
            want:      1073.64,
        },
        {
            name:      "zero rate",
            principal: 100000,
            rate:      0,
            months:    12,
            want:      8333.33,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := CalculateMonthlyPayment(tt.principal, tt.rate, tt.months)
            if math.Abs(got-tt.want) > 0.01 {
                t.Errorf("CalculateMonthlyPayment() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

## Performance
- Use `go build -race` to detect race conditions
- Profile with `go tool pprof` for performance bottlenecks
- Use `sync.Pool` for expensive object reuse
- Prefer slices over arrays for flexibility
- Use `strings.Builder` for efficient string concatenation

```go
// Good: Efficient string building
func BuildQuery(fields []string, table string) string {
    var builder strings.Builder
    builder.WriteString("SELECT ")
    for i, field := range fields {
        if i > 0 {
            builder.WriteString(", ")
        }
        builder.WriteString(field)
    }
    builder.WriteString(" FROM ")
    builder.WriteString(table)
    return builder.String()
}
```

## Dependencies and Modules
- Use Go modules for dependency management
- Keep dependencies minimal and well-maintained
- Use `go mod tidy` to clean up unused dependencies
- Pin dependency versions in production
- Use `go mod vendor` for reproducible builds

```bash
# Initialize module
go mod init github.com/username/project

# Add dependency
go get github.com/gorilla/mux@v1.8.0

# Clean up
go mod tidy

# Vendor dependencies
go mod vendor
```