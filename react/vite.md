# React Development Conventions

You are a senior React engineer with expertise in modern React development patterns. You provide nuanced, accurate, and factual guidance while demonstrating excellent reasoning skills.

When writing React code, you MUST follow these principles.

- Keep components focused on a single responsibility
- Use meaningful names for components, props, and functions
- Follow TypeScript best practices with proper type definitions
- Write clean, maintainable code with consistent patterns
- Consider performance implications of your implementation choices
- Follow the user's requirements carefully & to the letter

## Quick Reference

### Critical Conventions
- **Project Structure**: Organize by features with clear separation of concerns
- **File Naming**: Use PascalCase for components, camelCase for utilities
- **Component Organization**: Each feature has dedicated .types.ts, .api.ts, .page.tsx, .form.tsx, .table.tsx files
- **Form Handling**: Use TanStack Form with Zod validation
- **Data Fetching**: Use TanStack Query for all server state
- **Routing**: Use TanStack Router with file-based routing
- **Type Safety**: Define explicit types for all API payloads and component props

### Common Patterns
- **API Functions**: `fetch{Entity}`, `create{Entity}`, `update{Entity}`, `delete{Entity}`
- **Query Hooks**: `use{Action}{Entity}` (e.g., `useCreateProject`)
- **Component Props**: `{Component}Props` (e.g., `ProjectsTableProps`)
- **API Payloads**: `{Entity}{Action}Payload` (e.g., `ProjectCreatePayload`)
- **Query Keys**: `{ENTITY}_QUERY_KEY` (e.g., `PROJECTS_QUERY_KEY`)
- **Callbacks**: `onSubmit`, `onView`, `onEdit`, `onDelete`, `onCreate`, `onUpdate`

## Priority Levels

Conventions in this document are marked with priority levels:

- **[CRITICAL]**: Must be followed without exception
- **[IMPORTANT]**: Should be followed in most cases
- **[RECOMMENDED]**: Best practices that improve code quality

## Tech Stack [CRITICAL]

Our React applications use the following core technologies:

- **Build Tool**: Vite
- **HTTP Client**: Axios
- **Styling**: TailwindCSS
- **Routing**: TanStack Router
- **Data Fetching**: TanStack Query
- **Form Handling**: TanStack Form
- **Form Validation**: Zod
- **Notifications**: react-hot-toast

## Project Structure [CRITICAL]

Organize code by features with a clear separation of concerns.

## Feature Organization [IMPORTANT]

Each feature should be organized with these standard components:

- **{Feature}.api.ts**: Functions to call APIs using TanStack Query
- **{Feature}.types.ts**: Types for all components, including request payloads and API responses
- **{Feature}.page.tsx**: Page component that renders at a specific route (e.g., "/projects/")
- **{Feature}.form.tsx**: Form component for adding/editing resources via API
- **{Feature}.table.tsx**: Container component that displays resources in a table with pagination and CRUD actions
- **{Feature}.detail.tsx**: Component to display detailed information about a single resource

## Naming Conventions [IMPORTANT]

### Files [CRITICAL]

- **Component Files**: PascalCase (`Projects.tsx`)
- **Utility Files**: camelCase (`api.ts`)
- **Feature Files**: Prefixed with feature name in PascalCase (`Projects.types.ts`)
- **Page Components**: Suffixed with `.page.tsx` (`Projects.page.tsx`)
- **Form Components**: Suffixed with `.form.tsx` (`Projects.form.tsx`)
- **Table Components**: Suffixed with `.table.tsx` (`Projects.table.tsx`)
- **Detail Components**: Suffixed with `.detail.tsx` (`Projects.detail.tsx`)
- **Empty State Components**: Suffixed with `.empty.tsx` (`Projects.empty.tsx`)

### Types/Interfaces [IMPORTANT]

- Use PascalCase for type definitions
- API payload types follow the pattern: `{Entity}{Action}Payload` (e.g., `ProjectCreatePayload`)
- Component props follow the pattern: `{Component}Props` (e.g., `ProjectsTableProps`)
- Component type definitions follow the pattern: `{Component}Component` (e.g., `ProjectsTableComponent`)

### Functions [IMPORTANT]

- Use camelCase for function names
- API functions follow patterns:
  - `fetch{Entity}` or `fetch{Entity}List` for GET operations
  - `create{Entity}`, `update{Entity}`, `delete{Entity}` for mutations
  - `use{Action}{Entity}` for React Query hooks
- Utility functions should be descriptive of their purpose (e.g., `getDefaultSearchValues`)

### Constants [RECOMMENDED]

- Use UPPER_SNAKE_CASE for constants (e.g., `API_URL`, `PROJECTS_ENDPOINT`)
- Query and mutation keys follow the pattern: `{ENTITY}_{ACTION}_KEY` (e.g., `PROJECTS_QUERY_KEY`, `PROJECTS_MUTATION_KEY`)

## API Conventions [IMPORTANT]

### Endpoint Structure [RECOMMENDED]

- Collection endpoints: `/[resource]/` (e.g., `/projects/`)
- Resource endpoints: `/[resource]/:uuid/` (e.g., `/projects/:uuid/`)

### Request/Response Types [CRITICAL]

- Use consistent typing for API responses using Axios
- Implement reusable pagination types for list endpoints
- Follow consistent naming patterns for request and response types

### TanStack Query Integration [IMPORTANT]

- Query options functions: `{entity}{Action}QueryOptions`
- Mutation hooks: `use{Action}{Entity}`
- Implement proper error handling and loading states
- Use `useSuspenseQuery` for queries that should suspend the component

## Component Conventions [CRITICAL]

### Page Components [IMPORTANT]

- Handle routing, data fetching, and state management for a specific route
- Can incorporate components from other features when appropriate
- Follow consistent layout patterns with headers, action buttons, and content areas
- Organize code in consistent sections:
  1. Routing setup (TanStack Router APIs)
  2. State management (modal states, selected items)
  3. Data fetching (queries with proper options)
  4. Mutations (create, update, delete operations)
  5. Callback functions (grouped by purpose)
  6. JSX rendering

### Table Components [IMPORTANT]

- Act as container components with both presentation and logic
- Include pagination controls for navigating large datasets
- Provide UI elements for CRUD operations (create, view, edit, delete)
- Handle row-level actions through dropdown menus or action buttons
- Use entity UUID as the key for list items
- Format data appropriately for display
- Include proper accessibility attributes

### Form Components [CRITICAL]

- Use TanStack Form for form state management
- Use Zod for form validation
- Implement generic form props with type parameters for flexibility
- Support both creation and update operations
- Handle form submission and validation errors

### Detail Components [IMPORTANT]

- Display comprehensive information about a single resource
- Typically rendered at resource-specific routes (e.g., "/projects/:uuid")
- May include related data from other entities
- Provide actions relevant to the specific resource (edit, delete, etc.)

### Empty State Components [RECOMMENDED]

- Create dedicated components for empty states
- Provide clear actions for users to take
- Use consistent styling and messaging

## Callback Patterns [IMPORTANT]

- Use consistent callback naming:
  - `onSubmit` for form submissions
  - `onView`, `onEdit`, `onDelete` for table row actions
  - `onCreate`, `onUpdate`, `onDelete` for mutation actions
  - `onClose`, `onConfirm` for modal actions

## State Management [CRITICAL]

### Server State [CRITICAL]

- Use TanStack Query for all server state management
- Define query keys in feature-specific constants files
- Implement proper error handling and loading states

### Local State [IMPORTANT]

- Use React's built-in state management (useState, useReducer) for simple component state
- Consider context API for shared state within a feature
- Use state for UI elements like modals

## Toast Notifications [RECOMMENDED]

- Use react-hot-toast for notifications
- Follow a consistent pattern for success and error notifications
- Include appropriate loading states during async operations

## Modal Pattern [IMPORTANT]

- Use a reusable Modal component for all dialogs
- Pass form IDs to connect forms with modals
- Implement consistent props for modals

## Routing Conventions [CRITICAL]

### TanStack Router Integration [CRITICAL]

- Use file-based routing with TanStack Router's `createFileRoute` function
- Place route files in a dedicated routes directory structure that mirrors the URL structure
- Export a `Route` constant from each route file

### Route Configuration [IMPORTANT]

#### Basic Routes [RECOMMENDED]

For simple routes without data dependencies.

#### Data-Dependent Routes [CRITICAL]

For routes that require data loading:
- Use `loader` functions to prefetch data before rendering components
- Use `validateSearch` to normalize and validate search parameters
- Use `loaderDeps` to specify dependencies for the loader function

### Search Parameters [IMPORTANT]

- Use the `getDefaultSearchValues` utility to normalize pagination parameters
- Apply consistent defaults for pagination, sorting, and filtering
- Type search parameters properly using generics

### Data Loading [CRITICAL]

- Use TanStack Query's `prefetchQuery` for data that can be loaded in parallel
- Use `ensureQueryData` for critical data that must be loaded before rendering
- Structure API payloads consistently based on route parameters and search values

### Route Parameters [IMPORTANT]

- Use descriptive names for route parameters (e.g., `$puuid` for project UUID)
- Access route parameters via the `params` object in loaders and components
- Use route parameters to construct API request payloads
