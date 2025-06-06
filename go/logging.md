# Go Logging Conventions

How we handle structured logging in Go applications.

## Library

We use **zerolog** for all structured logging:

```go
import "github.com/rs/zerolog"
```

## Handler Pattern

Always inject logger into handlers via constructor:

```go
type Handler struct {
    service Service
    logger  *zerolog.Logger
}

func NewHandler(service Service, logger *zerolog.Logger) *Handler {
    return &Handler{
        service: service,
        logger:  logger,
    }
}
```

## Log Levels

Use these levels consistently:

- **Debug**: Operation start with parameters
- **Info**: Successful operations with key identifiers
- **Error**: Failures with error details

## Message Patterns

### Debug Messages
Log operation start with user context:

```go
h.logger.Debug().
    Interface("params", userData).
    Msg("account creation")
```

### Info Messages
Log successful operations with resource identifiers:

```go
h.logger.Info().Str("uuid", account.UUID).Msg("account created")
h.logger.Info().Msg("accounts listed")
```

### Error Messages
Log failures with error and descriptive message:

```go
h.logger.Error().Err(err).Msg("unable to create account")
```

## Field Naming

- Use `uuid` for resource identifiers
- Use `params` for user/request data
- Use `Err(err)` for error objects
- Use `Interface()` for complex objects
- Use `Str()` for string values

## Message Format

Keep messages:
- Lowercase
- Action-focused ("account created", "unable to create account")
- Consistent across similar operations
- Descriptive but concise

## CRUD Operations

Follow this pattern for standard CRUD operations:

```go
// GET
h.logger.Debug().Interface("params", userData).Msg("account retrieval")
h.logger.Info().Str("uuid", input.UUID).Msg("account found")

// CREATE
h.logger.Debug().Interface("params", userData).Msg("account creation")
h.logger.Info().Str("uuid", account.UUID).Msg("account created")

// LIST
h.logger.Debug().Interface("params", userData).Msg("accounts listing")
h.logger.Info().Msg("accounts listed")

// UPDATE
h.logger.Debug().Interface("params", userData).Msg("account updating")
h.logger.Info().Str("uuid", account.UUID).Msg("account updated")

// DELETE
h.logger.Debug().Interface("params", userData).Msg("account deletion")
h.logger.Info().Str("uuid", input.UUID).Msg("account deleted")
```

## Error Handling

Always log errors before returning HTTP responses:

```go
account, err := h.service.GetAccount(input.UUID, input.LedgerUUID, userID)
if err != nil {
    h.logger.Error().Err(err).Msg("unable to get account")
    // handle HTTP response
}
```

## What NOT to Log

- Sensitive data (passwords, tokens)
- Large payloads in production
- Redundant success messages
