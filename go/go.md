# Go General Conventions

General Go conventions that apply to all Go projects (API services, CLI apps, libraries).

Follow the patterns in [Effective Go](https://go.dev/doc/effective_go) as a baseline, but don't be dogmatic—pragmatism over purity.

## Related Conventions

- **[api.md](api.md)** - API services (handlers, routing, middleware)
- **[cli.md](cli.md)** - CLI applications (cobra, flags, output)
- **[logging.md](logging.md)** - Structured logging with zerolog
- **[examples.md](examples.md)** - Reference code snippets

## General Code Quality

- Code should be easy to read and understand
- Keep the code as simple as possible. Avoid unnecessary complexity
- Use meaningful names for variables, functions, etc. Names should reveal intent while following Go conventions
- Functions should be small and do one thing well
- Prefer fewer arguments in functions. Ideally, aim for no more than two or three arguments
- Only use comments when necessary, as they can become outdated. Instead, strive to make the code self-explanatory
- When comments are used, they should add useful information that is not readily apparent from the code itself
- Consider security implications of the code. Implement security best practices

## Project Structure

### Standard Directory Layout

```
go-project/
├── cmd/
│   └── main/           # Main application entry points
│       └── main.go
├── internal/           # Private application code
│   ├── api/           # API-specific code (for API services)
│   ├── config/        # Configuration management
│   └── pkg/           # Internal packages
├── pkg/               # Public library code
├── go.mod
└── go.sum
```

### File Naming Conventions

- Use lowercase with underscores for multi-word files: `user_service.go`
- Group related functionality in packages, not files
- Avoid generic names like `utils.go` or `helpers.go`

## Configuration Management

### Environment-based Config with envconfig

Use the `envconfig` package for clean environment variable handling:

```go
package config

import (
	"github.com/kelseyhightower/envconfig"
)

type Config struct {
	Env  string `envconfig:"ENV" default:"dev"`
	Port string `envconfig:"PORT" default:"8080"`
}

// LoadEnv loads the configuration from environment variables
func (c Config) LoadEnv() (Config, error) {
	cfg := c
	if err := envconfig.Process("", &cfg); err != nil {
		return c, err
	}
	return cfg, nil
}

func New() (*Config, error) {
	var cfg Config
	loadedCfg, err := cfg.LoadEnv()
	if err != nil {
		return nil, err
	}
	return &loadedCfg, nil
}
```

## Constructor Patterns

### Simple Constructor Pattern

Use this pattern for creating service components with required dependencies:

```go
package service

type Config struct {
    store  Store
    logger *zerolog.Logger // Always include logger in all layers
}

// Implementation is lowercase (unexported)
type service struct {
    cfg Config
}

// Constructor returns the interface type
func NewService(store Store, logger *zerolog.Logger) Service {
    return &service{
        cfg: Config{
            store:  store,
            logger: logger,
        },
    }
}
```

**When to use:**
- Required dependencies only (2-3 parameters maximum)
- Most common pattern in Go
- Clear and straightforward

### Builder Pattern (Fluent Interface)

Use this pattern when you need step-by-step configuration with method chaining:

```go
type Config struct {
    listenAddr string
    id         string
    name       string
}

func (c Config) WithName(name string) Config {
    c.name = name
    return c
}

func (c Config) WithID(id string) Config {
    c.id = id
    return c
}

func (c Config) WithListenAddr(addr string) Config {
    c.listenAddr = addr
    return c
}

func NewConfig() Config {
    return Config{
        id:         "randomuuid",
        name:       "defaultName",
        listenAddr: ":3000",
    }
}

// Usage
config := NewConfig().
    WithID("longuuid").
    WithName("my server").
    WithListenAddr(":8000")
```

**When to use:**
- Complex configuration objects
- Step-by-step construction needed
- Internal APIs where you control usage
- When order of configuration matters

### Functional Options Pattern

Use this pattern for functions with many optional parameters, especially in public APIs:

```go
type Config struct {
    port    int
    host    string
    timeout time.Duration
    debug   bool
}

type Server struct {
    config Config
}

// Option is a function that modifies Config
type Option func(*Config)

// Option functions
func WithPort(port int) Option {
    return func(c *Config) {
        c.port = port
    }
}

func WithHost(host string) Option {
    return func(c *Config) {
        c.host = host
    }
}

func WithTimeout(timeout time.Duration) Option {
    return func(c *Config) {
        c.timeout = timeout
    }
}

func WithDebug(debug bool) Option {
    return func(c *Config) {
        c.debug = debug
    }
}

// Constructor accepts variadic options
func NewServer(opts ...Option) (*Server, error) {
    // Set defaults
    config := Config{
        port:    8080,
        host:    "localhost",
        timeout: 30 * time.Second,
        debug:   false,
    }
    
    // Apply options
    for _, opt := range opts {
        opt(&config)
    }
    
    return &Server{config: config}, nil
}

// Usage
server, _ := NewServer(
    WithPort(9000),
    WithHost("0.0.0.0"),
    WithDebug(true),
    // timeout uses default
)
```

**When to use:**
- Public library APIs
- Many optional parameters (4+)
- Want to follow Go idioms (used in stdlib)
- Need to provide sensible defaults
- Options might be reused/composed

## Error Handling

### Custom Error Types

Define domain-specific errors for better error handling:

```go
var (
    ErrNotFound = errors.New("resource not found")
    ErrUnauthorized = errors.New("unauthorized access")
    ErrInvalidInput = errors.New("invalid input")
)

// Use errors.Is for checking
if errors.Is(err, ErrNotFound) {
    // handle not found
}
```

### Error Wrapping

Use `fmt.Errorf` with `%w` verb to wrap errors:

```go
if err != nil {
    return fmt.Errorf("unable to get account with UUID %s: %w", uuid, err)
}
```

### Handle Errors Once

- If the error is recoverable, log it and degrade gracefully
- If the error represents a domain-specific failure, return a well-defined error
- Otherwise, return the error (wrapped or verbatim)
- **Important**: Handle each error only once. Don't log an error and then return it

## Context Handling

- Always pass `context.Context` as the first parameter in service and store methods
- Use `context.Background()` sparingly - prefer passing context from the request
- Set appropriate timeouts for database operations
- Use context for cancellation and request tracing

```go
// Good: Pass context from request
func (s *service) GetAccount(ctx context.Context, uuid string, userID int64) (*GetOutput, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    // ... implementation
}
```

## Logging Conventions

### Structured Logging

Always use structured logging with consistent field names:

```go
logger.Debug().
    Str("resource_uuid", input.UUID).
    Str("user_uuid", userData.UUID).
    Str("operation", "account_retrieval").
    Msg("processing request")

// Error messages should include the resource identifier
logger.Error().Err(err).
    Str("resource_uuid", input.UUID).
    Msg("unable to get account")
```

### Log Levels

- **Debug**: Request processing, successful operations
- **Info**: Important business events (rare)
- **Warn**: Recoverable errors, degraded functionality
- **Error**: Unrecoverable errors that need attention

## Return Types

### Single Objects

Return pointers (`*Type`) when:
- The object might not exist (nil can represent "not found")
- The object is large or contains reference types
- The object might be modified by the caller

```go
// Example: Single object returns use pointers
func GetResource(id string) (*Resource, error) {
  if notFound {
    return nil, errors.New("not found")
  }
  return &Resource{...}, nil
}
```

### Collections

Return slices of values (`[]Type`) when:
- Returning multiple items
- The items won't be modified individually by the caller

```go
// Example: Collections return slices of values, not pointers
func ListResources() ([]Resource, error) {
  if noResults {
    return []Resource{}, nil  // Empty slice, not nil
  }
  return []Resource{{...}, {...}}, nil
}
```

## Security Conventions

- Never log sensitive information (passwords, API keys, tokens)
- Use user UUIDs in logs, never internal IDs
- Validate user ownership of resources before operations
- Implement rate limiting for public endpoints
- Use HTTPS in production
- Validate and sanitize all inputs
- Use parameterized queries

## Testing Conventions

### Test File Structure

- `*_test.go` files in the same package
- Use descriptive test names: `TestService_GetAccount_Success`
- Use table-driven tests for multiple scenarios

### Test Naming

```go
func TestService_GetAccount_Success(t *testing.T) {}
func TestService_GetAccount_NotFound(t *testing.T) {}
func TestHandler_Create_ValidationError(t *testing.T) {}
```