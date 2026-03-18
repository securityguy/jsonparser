# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

This is a **released, public library** (`github.com/securityguy/jsonparser`). All changes must maintain backwards compatibility.

## Commands

```bash
make test        # Run tests (10s timeout)
make race        # Run tests with race detector
make bench       # Run benchmarks
make fmt         # Format code
make vet         # Go vet
```

Run a single test:
```bash
go test -v -run TestGetString .
```

## Architecture

**jsonparser** is a high-performance, zero-allocation JSON parser for Go. It provides direct field access by key path without requiring Go structs.

### Core Files

- `parser.go` — Public API and main parsing logic (`Get`, `Set`, `Delete`, `ArrayEach`, `ObjectEach`, `EachKey`)
- `escape.go` — Unicode/UTF-16 escape sequence decoding
- `bytes.go` — Custom integer/float parsing (~2x faster than `strconv` for JSON numbers)
- `bytes_unsafe.go` / `bytes_safe.go` — Platform-specific byte↔string conversions; `bytes_unsafe.go` is used by default (build tag `!appengine`), `bytes_safe.go` for AppEngine (`appengine` tag)

### Key Design Principles

- **Zero allocation**: The parser avoids heap allocations on hot paths. Avoid changes that introduce allocations in `Get`, `ArrayEach`, `ObjectEach`, or `EachKey`.
- **No schema**: Returns `[]byte` slices into the original data; callers convert types via helpers like `GetString`, `GetInt`, `GetFloat`.
- **`EachKey` is the fastest multi-path API**: It scans the JSON once for multiple keys simultaneously.

### Value Types

```go
const (
    NotExist ValueType = iota
    String
    Number
    Object
    Array
    Boolean
    Null
    Unknown
)
```

### Benchmarks

The `benchmark/` directory has its own `go.mod` and compares against ffjson, easyjson, and codecgen. Run separately with `cd benchmark && go test -bench=.`.
