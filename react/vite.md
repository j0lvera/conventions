# React Conventions

This document outlines our React development conventions when using Vite as the build tool.

## Tech Stack

- **Build Tool**: Vite
- **HTTP Client**: Axios
- **Styling**: TailwindCSS
- **Routing**: TanStack Router
- **Data Fetching**: TanStack Query
- **Form Handling**: TanStack Form

## Project Structure

```
/src
  /components
    /[feature]
      [Feature].tsx         # Main component
      [Feature].types.ts    # Type definitions
      [Feature].api.ts      # API integration
      [Feature].constants.ts # Constants
      [Feature].utils.ts    # Utility functions
      /components           # Feature-specific components
  /api.ts                   # Global API setup
  /constants.ts             # Global constants
  /types.ts                 # Global type definitions
  /utils.ts                 # Global utility functions
```

## Naming Conventions

### Files

- **Component Files**: PascalCase (`Projects.tsx`)
- **Utility Files**: camelCase (`api.ts`)
- **Feature Files**: Prefixed with feature name in PascalCase (`Projects.types.ts`)

### Types/Interfaces

- Use PascalCase for type definitions
- API payload types follow the pattern: `{Entity}{Action}Payload` (e.g., `ProjectCreatePayload`)
- Component props follow the pattern: `{Component}Props` (e.g., `ProjectsTableProps`)
- Component type definitions follow the pattern: `{Component}Component` (e.g., `ProjectsTableComponent`)

### Functions

- Use camelCase for function names
- API functions follow patterns:
  - `fetch{Entity}` or `fetch{Entity}List` for GET operations
  - `create{Entity}`, `update{Entity}`, `delete{Entity}` for mutations
  - `use{Action}{Entity}` for React Query hooks
- Utility functions should be descriptive of their purpose (e.g., `getDefaultSearchValues`)

### Constants

- Use UPPER_SNAKE_CASE for constants (e.g., `API_URL`, `PROJECTS_ENDPOINT`)

## API Conventions

### API Setup

```typescript
// api.ts
import axios from "axios";
import { API_URL } from "@/constants.ts";

const api = axios.create({
  baseURL: API_URL,
  withCredentials: true,
});

export { api };
```

### Endpoint Structure

- Collection endpoints: `/[resource]/` (e.g., `/projects/`)
- Resource endpoints: `/[resource]/:uuid/` (e.g., `/projects/:uuid/`)

### Request/Response Types

- Use consistent typing for API responses using Axios
- Implement reusable pagination types:

```typescript
type PaginatedRes<T> = {
  items: T[];
  limit: number;
  offset: number;
  total: number;
};

interface PaginationReq<T> {
  // Pagination
  limit: number;
  offset: number;
  // Sorting
  sort_by: keyof T;
  sort_direction: "asc" | "desc";
  // Filtering
  filter: string;
  // Date range
  start_date?: string;
  end_date?: string;
}
```

### TanStack Query Integration

- Query options functions: `{entity}{Action}QueryOptions`
- Mutation hooks: `use{Action}{Entity}`

```typescript
// Example query options
const projectListQueryOptions = (payload: ProjectListGetPayload) => {
  return queryOptions({
    queryKey: [PROJECTS_QUERY_KEY, payload],
    queryFn: () => fetchProjectList(payload),
  });
};

// Example mutation hook
const useCreateProject = () => {
  return useMutation({
    mutationKey: [PROJECTS_MUTATION_KEY],
    mutationFn: (payload: ProjectCreatePayload) => createProject(payload),
  });
};
```

## Component Conventions

### Props Typing

- Define explicit interfaces for component props
- Use reusable types for common patterns (forms, tables)

```typescript
interface ProjectsTableProps {
  data: Project[];
  onView?: (project: Project) => void;
  onEdit?: (project: Project) => void;
  onDelete?: (project: Project) => void;
}

type ProjectsTableComponent = (
  props: ProjectsTableProps
) => ReactElement | null;
```

### Form Handling

- Use generic form props with type parameters for flexibility:

```typescript
interface ProjectsFormProps<
  T extends ProjectCreatePayload | ProjectUpdatePayload,
> {
  formId?: string;
  onSubmit?: (data: T) => void;
  defaultValues?: T;
  isLoading?: boolean;
  hideActions?: boolean;
}
```

### Callback Patterns

- Use consistent callback naming:
  - `onSubmit` for form submissions
  - `onView`, `onEdit`, `onDelete` for table row actions

## Utility Functions

- Create reusable utility functions for common operations:

```typescript
const getDefaultSearchValues = <T>(search?: Partial<PaginationReq<T>>) => ({
  offset: Number(search?.offset ?? 0),
  limit: Number(search?.limit ?? 10),
  filter: String(search?.filter ?? ""),
  sort_by: (search?.sort_by ?? "name") as keyof T,
  sort_direction: (search?.sort_direction ?? "asc") as "asc" | "desc",
});
```

## State Management

### Server State

- Use TanStack Query for all server state management
- Define query keys in feature-specific constants files
- Implement proper error handling and loading states

### Local State

- Use React's built-in state management (useState, useReducer) for simple component state
- Consider context API for shared state within a feature
