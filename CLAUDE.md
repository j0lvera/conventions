# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains development conventions and code style guides for AI coding assistants. It's organized by language/framework with detailed conventions for Go, Python/FastAPI, React, and PostgreSQL.

## Development Commands

The project uses Task for build automation. Key commands:

```bash
# AI Assistant Access
task ai          # Run Anthropic Claude Sonnet 4 (default)
task ai:r1       # Run DeepSeek R1 assistant
task ai:gemini   # Run Gemini 2.5 Pro assistant
```

These commands use aider for AI-assisted development with different models via OpenRouter.

## Repository Structure

```
src/conventions/
├── go/                     # Go conventions
│   ├── go.md              # General Go patterns & architecture
│   ├── api.md             # API service conventions
│   ├── cli.md             # CLI application patterns
│   ├── logging.md         # Structured logging practices
│   └── examples.md        # Code examples
├── python/                # Python conventions
│   └── fastapi/
│       ├── fastapi.md     # FastAPI conventions & architecture
│       ├── logging.md     # Python logging practices
│       └── examples.md    # Code examples
├── react/                 # React conventions
│   ├── vite.md           # React with Vite conventions
│   └── examples.md       # React examples
├── postgres/              # Database conventions
│   └── tables.md         # PostgreSQL schema patterns
├── README.md             # Project overview
├── CRUSH.md              # Quick reference guide
└── Taskfile.yml          # Build commands
```

## Architecture Patterns

### Go Applications
- **Layered Architecture**: Handler → Service → Store
- **File Organization**: `handler.go`, `service.go`, `store.go`, `types.go`
- **Error Handling**: Wrap errors with context, handle once
- **Logging**: Structured logging with zerolog
- **Configuration**: Use envconfig with `LoadEnv()` pattern

### Python/FastAPI Applications
- **Repository Pattern**: Store → Service → Router
- **Method Naming**: `get_one`, `get_plist`, `create_one`, `update_*`, `delete_one`
- **Type Safety**: Payload objects over individual parameters
- **Exception Flow**: Store propagates DB exceptions, Service transforms to domain exceptions
- **Response Models**: Always return Pydantic models, not ORM objects

### React Applications
- **Tech Stack**: Vite + TanStack Router/Query/Form + Zod + TailwindCSS
- **Organization**: Feature-based in `src/features/`, pages in `src/Pages/`
- **File Conventions**: `{Feature}.api.ts`, `{Feature}.types.ts`, `{Feature}.form.tsx`
- **API Patterns**: `{Entity}ListGetPayload`, `use{Action}{Entity}` hooks
- **Form Handling**: TanStack Form with Zod validation

### PostgreSQL Schemas
- **Naming**: snake_case for all database objects
- **Standard Fields**: `id` (bigint), `uuid` (text nanoid), `created_at`/`updated_at` (timestamptz)
- **Constraints**: Named explicitly with pattern `{table}_{column}_{type}`
- **Data Types**: Use `text` over `varchar`, `timestamptz` for timestamps, `jsonb` for JSON

## Key Conventions Summary

### Security & Best Practices
- Never expose internal database IDs - use UUIDs for client-facing identifiers
- Validate user ownership before any resource operations
- Use parameterized queries to prevent SQL injection
- Implement structured logging with consistent field names
- Handle errors once - avoid logging and then returning the same error

### Code Quality Guidelines
- Keep code simple and readable - avoid unnecessary complexity
- Use meaningful names that reveal intent
- Functions should be small and focused on single responsibility
- Include proper type hints and validation
- Write self-documenting code, minimize comments

### Testing Approach
- Write unit tests for business logic
- Use descriptive test names: `TestService_GetAccount_Success`
- Mock external dependencies in tests
- Test error conditions and edge cases

## Important Files to Reference

When working on specific technologies, refer to these detailed convention files:

- **Go Development**: `go/go.md` for general patterns, `go/api.md` for API services
- **Python Development**: `python/fastapi/fastapi.md` for comprehensive FastAPI patterns
- **React Development**: `react/vite.md` for complete React conventions
- **Database Work**: `postgres/tables.md` for PostgreSQL schema design
- **Quick Reference**: `CRUSH.md` for condensed guidelines across all technologies

## Development Workflow

1. Read the relevant convention file for your technology stack
2. Follow the architectural patterns and naming conventions
3. Use the provided examples as templates for new code
4. Run appropriate linting/testing commands before committing
5. Use structured logging and proper error handling patterns

This repository serves as the authoritative source for how code should be structured and written across all supported technologies and frameworks.