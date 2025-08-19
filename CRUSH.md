# CRUSH.md - Development Conventions

## Build/Test Commands
- `task ai` - Run Anthropic assistant with Claude Sonnet 4
- `task ai:r1` - Run DeepSeek R1 assistant  
- `task ai:gemini` - Run Gemini 2.5 Pro assistant

## Code Style Guidelines

### Go Conventions
- Use clean architecture: Handler → Service → Store layers
- File naming: `handler.go`, `service.go`, `store.go`, `types.go`
- Return pointers for single objects, slices for collections
- Use structured logging with zerolog: `logger.Debug().Str("resource_uuid", uuid).Msg("resource creation")`
- Error handling: wrap with `fmt.Errorf("unable to get resource: %w", err)`
- Use envconfig for configuration with `LoadEnv()` method pattern

### Python/FastAPI Conventions  
- Architecture: Store → Service → Router pattern
- Method naming: `get_one`, `get_plist`, `create_one`, `update_*`, `delete_one`
- Use payload objects instead of individual parameters
- Exception handling: Store lets DB exceptions propagate, Service transforms to domain exceptions
- Type hints: Use `list[T]` over `List[T]`, consistent async/await

### React Conventions
- Tech stack: Vite + TanStack Router/Query/Form + Zod + TailwindCSS
- Structure: Feature-based organization in `src/features/`, pages in `src/Pages/`
- File naming: `{Feature}.api.ts`, `{Feature}.types.ts`, `{Feature}.form.tsx`, `{Feature}.table.tsx`
- API types: `{Entity}ListGetPayload`, `{Entity}CreatePayload`, `{Entity}Response`
- Query hooks: `use{Action}{Entity}`, query options: `{entity}{Action}QueryOptions`

### PostgreSQL Conventions
- Use snake_case for all database objects
- Standard fields: `id` (bigint identity), `uuid` (text default nanoid(8)), `created_at`/`updated_at` (timestamptz)
- Constraints: Named explicitly `{table}_{column}_{type}`
- Always add `updated_at` trigger using `set_updated_at_fn()`
- Use `text` over `varchar`, `timestamptz` for timestamps, `jsonb` for semi-structured data

## Security & Best Practices
- Never expose internal IDs to clients, use UUIDs
- Validate user ownership before operations
- Use parameterized queries, structured logging
- Handle errors once - don't log and return
- Follow principle of least privilege