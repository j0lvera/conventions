# React Conventions

This document outlines our React development conventions when using Vite as the build tool.

## Tech Stack

- **Build Tool**: Vite
- **HTTP Client**: Axios
- **Styling**: TailwindCSS
- **Routing**: TanStack Router
- **Data Fetching**: TanStack Query
- **Form Handling**: TanStack Form
- **Form Validation**: Zod
- **Notifications**: react-hot-toast

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
      [Feature].page.tsx    # Page component
      [Feature].form.tsx    # Form component
      [Feature].table.tsx   # Table component
      [Feature].empty.tsx   # Empty state component
      /components           # Feature-specific components
  /components
    /ui                     # Shared UI components
    /common                 # Shared common components
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
- **Page Components**: Suffixed with `.page.tsx` (`Projects.page.tsx`)
- **Form Components**: Suffixed with `.form.tsx` (`Projects.form.tsx`)
- **Table Components**: Suffixed with `.table.tsx` (`Projects.table.tsx`)
- **Empty State Components**: Suffixed with `.empty.tsx` (`Projects.empty.tsx`)

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
- Query and mutation keys follow the pattern: `{ENTITY}_{ACTION}_KEY` (e.g., `PROJECTS_QUERY_KEY`, `PROJECTS_MUTATION_KEY`)

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

### Page Components

- Page components handle routing, data fetching, and state management
- Follow the pattern: `{Feature}Page` (e.g., `ProjectsPage`)
- Use TanStack Router for navigation and search params
- Implement proper loading and error states

```typescript
const ProjectsPage = () => {
  // Routing
  const routeApi = getRouteApi("/_app/p/");
  const search = routeApi.useSearch();
  const navigate = routeApi.useNavigate();
  
  // Data fetching
  const projectsQuery = useSuspenseQuery(
    projectListQueryOptions(projectListPayload)
  );
  
  // Component state
  const [isDeleteModalOpen, setIsDeleteModalOpen] = useState(false);
  
  // Event handlers
  const onTableItemView = async (project: Project) => {
    await navigate({
      to: "/p/$puuid",
      params: { puuid: project.uuid },
      search: { /* search params */ },
    });
  };
  
  return (
    // JSX
  );
};
```

### Table Components

- Table components display data in a structured format with consistent styling
- Follow the pattern: `{Feature}Table` (e.g., `ProjectsTable`)
- Use shared UI table components (`Table`, `TableHead`, `TableRow`, etc.)
- Implement standardized props for data and callbacks

```typescript
const ProjectsTable: ProjectsTableComponent = ({
  data,
  onView,
  onEdit,
  onDelete,
}) => {
  // Event handlers
  const handleOnDelete = (
    event: MouseEvent<HTMLButtonElement>,
    project: Project
  ) => {
    event.stopPropagation();
    onDelete?.(project);
  };

  return (
    <Table>
      <TableHead>
        <TableRow>
          <TableHeader>Name</TableHeader>
          <TableHeader>URL</TableHeader>
          <TableHeader className="text-right">Actions</TableHeader>
        </TableRow>
      </TableHead>
      <TableBody>
        {data.map((project) => (
          <TableRow key={project.uuid}>
            <TableCell>{project.name}</TableCell>
            <TableCell>{project.url.replace("https://", "")}</TableCell>
            <TableCell className="text-right">
              <DropdownMenu>
                <DropdownMenuTrigger asChild>
                  <Button variant="ghost" size="icon">
                    <MoreHorizontal className="h-4 w-4" />
                    <span className="sr-only">Open menu</span>
                  </Button>
                </DropdownMenuTrigger>
                <DropdownMenuContent align="end">
                  <DropdownMenuItem onClick={() => onView?.(project)}>
                    View
                  </DropdownMenuItem>
                  <DropdownMenuItem onClick={() => onEdit?.(project)}>
                    Edit
                  </DropdownMenuItem>
                  <DropdownMenuSeparator />
                  <DropdownMenuItem
                    onClick={(e) => handleOnDelete(e, project)}
                    className="text-destructive"
                  >
                    Delete
                  </DropdownMenuItem>
                </DropdownMenuContent>
              </DropdownMenu>
            </TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
};
```

#### Table Component Conventions

- Use entity UUID as the key for list items
- Format data appropriately for display (e.g., URL formatting)
- Implement consistent action patterns with dropdown menus
- Use optional chaining for callback props (e.g., `onView?.(project)`)
- Prevent default behavior and stop propagation when needed
- Include proper accessibility attributes (e.g., `sr-only` for screen readers)
- Apply consistent TailwindCSS classes for styling

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

- Use TanStack Form for form state management
- Use Zod for form validation
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

- Implement form validation with Zod:

```typescript
<form.Field
  name="url"
  validators={{
    onChange: z.string().url("Invalid URL"),
    onBlur: z.string().url("Invalid URL"),
  }}
>
  {/* Field rendering */}
</form.Field>
```

### Callback Patterns

- Use consistent callback naming:
  - `onSubmit` for form submissions
  - `onView`, `onEdit`, `onDelete` for table row actions
  - `onCreate`, `onUpdate`, `onDelete` for mutation actions
  - `onClose`, `onConfirm` for modal actions

### Toast Notifications

- Use react-hot-toast for notifications
- Follow a consistent pattern for success and error notifications:

```typescript
let toastId = toast.loading("Creating project...");
createProject.mutate(payload, {
  onSuccess: () => {
    projectsQuery.refetch();
    setIsCreateModalOpen(false);
    toast.success("Project created successfully", { id: toastId });
  },
  onError: () => {
    toast.error("Unable to create project", { id: toastId });
  },
});
```

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
- Use `useSuspenseQuery` for queries that should suspend the component

### Local State

- Use React's built-in state management (useState, useReducer) for simple component state
- Consider context API for shared state within a feature
- Use state for UI elements like modals:

```typescript
const [isDeleteModalOpen, setIsDeleteModalOpen] = useState(false);
const [selectedProject, setSelectedProject] = useState<Project | null>(null);
```

## Modal Pattern

- Use a reusable Modal component for all dialogs
- Pass form IDs to connect forms with modals
- Implement consistent props for modals:

```typescript
<Modal
  formId="create-project-form"
  title={"Create a new project"}
  description={"Please enter the URL of the project you want to create."}
  isOpen={isCreateModalOpen}
  setIsOpen={setIsCreateModalOpen}
  onClose={() => setIsCreateModalOpen(false)}
  isLoading={createProject.isPending}
>
  <ProjectsForm
    hideActions
    formId="create-project-form"
    onSubmit={(values) => onCreate(values)}
    isLoading={false}
  />
</Modal>
```

## Empty States

- Create dedicated components for empty states
- Follow the pattern: `{Feature}Empty` (e.g., `ProjectsEmpty`)
- Provide clear actions for users to take
