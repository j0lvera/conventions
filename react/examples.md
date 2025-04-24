# React Code Examples

Use these examples as reference for implementing React conventions.

## API Setup

```typescript
// api.ts (global API setup)
import axios from "axios";
import { API_URL } from "@/constants.ts";

const api = axios.create({
  baseURL: API_URL,
  withCredentials: true,
});

export { api };
```

## Global Types

```typescript
// types.ts (global types)
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

export type { PaginationReq, PaginatedRes };
```

## Global Utils

```typescript
// utils.ts (global utils)
import { PaginationReq } from "./types.ts";

const getDefaultSearchValues = <T>(search?: Partial<PaginationReq<T>>) => ({
  offset: Number(search?.offset ?? 0),
  limit: Number(search?.limit ?? 10),
  filter: String(search?.filter ?? ""),
  sort_by: (search?.sort_by ?? "name") as keyof T,
  sort_direction: (search?.sort_direction ?? "asc") as "asc" | "desc",
});

export { getDefaultSearchValues };
```

## Global Constants

```typescript
// constants.ts (global constants)
// const API_URL = import.meta.env.VITE_API_URL
const API_URL = "/api";

export { API_URL };
```

## Feature Types

```typescript
// projects/Projects.types.ts
import type { ReactElement } from "react";
import { PaginationReq } from "@/types.ts";

type Project = {
  uuid: string;
  url: string;
  type: "WORDPRESS" | "SHOPIFY" | "CUSTOM";
};

// GET /projects/:uuid
interface ProjectGetPayload {
  uuid: string;
}

// GET /projects/
interface ProjectListGetPayload extends PaginationReq<Project> {}

// POST /projects/
interface ProjectCreatePayload extends Omit<Project, "uuid"> {
  url: string;
}

// PUT /projects/:uuid
type ProjectUpdatePayload = Partial<ProjectCreatePayload> &
  Pick<Project, "uuid">;

// Table types
interface ProjectsTableProps {
  data: Project[];
  onView?: (project: Project) => void;
  onEdit?: (project: Project) => void;
  onDelete?: (project: Project) => void;
}

type ProjectsTableComponent = (
  props: ProjectsTableProps
) => ReactElement | null;

// Form types
interface ProjectsFormProps<
  T extends ProjectCreatePayload | ProjectUpdatePayload,
> {
  formId?: string;
  onSubmit?: (data: T) => void;
  defaultValues?: T;
  isLoading?: boolean;
  hideActions?: boolean;
}

export type {
  Project,
  ProjectCreatePayload,
  ProjectUpdatePayload,
  ProjectGetPayload,
  ProjectListGetPayload,
  ProjectsTableComponent,
  ProjectsTableProps,
  ProjectsFormProps,
};
```

## Feature Constants

```typescript
// projects/Projects.constants.ts
const PROJECTS_ENDPOINT = "/projects/";

const PROJECTS_QUERY_KEY = "QUERY_PROJECTS";
const PROJECTS_MUTATION_KEY = "MUTATE_PROJECTS";

export { PROJECTS_ENDPOINT, PROJECTS_QUERY_KEY, PROJECTS_MUTATION_KEY };
```

## Feature API Integration

```typescript
// projects/Projects.api.ts
import { AxiosResponse } from "axios";
import {
  PROJECTS_ENDPOINT,
  PROJECTS_MUTATION_KEY,
  PROJECTS_QUERY_KEY,
} from "@/components/projects/Projects.constants.ts";
import {
  Project,
  ProjectCreatePayload,
  ProjectGetPayload,
  ProjectListGetPayload,
} from "@/components/projects/Projects.types.ts";
import { PaginatedRes } from "@/types.ts";
import { api } from "@/api.ts";
import { queryOptions, useMutation } from "@tanstack/react-query";

/**
 * Create a Project
 */

type CreateRes = AxiosResponse<Project>;
const createProject = async (payload: ProjectCreatePayload) => {
  const res: CreateRes = await api.post(PROJECTS_ENDPOINT, payload);
  return res.data;
};

const useCreateProject = () => {
  return useMutation({
    mutationKey: [PROJECTS_MUTATION_KEY],
    mutationFn: (payload: ProjectCreatePayload) => createProject(payload),
  });
};

/**
 * Get a Project
 * GET /projects/:uuid/
 */

type GetRes = AxiosResponse<Project>;

const fetchProject = async (payload: ProjectGetPayload) => {
  const res: GetRes = await api.get(PROJECTS_ENDPOINT + payload.uuid + "/");
  return res.data;
};

const projectGetQueryOptions = (payload: ProjectGetPayload) => {
  return queryOptions({
    queryKey: [PROJECTS_QUERY_KEY, payload],
    queryFn: () => fetchProject(payload),
  });
};

/**
 * Get a list of Projects
 * GET /projects/
 */

type ListRes = AxiosResponse<PaginatedRes<Project>>;

const fetchProjectList = async (payload: ProjectListGetPayload) => {
  const res: ListRes = await api.get(PROJECTS_ENDPOINT, {
    params: payload,
  });
  return res.data;
};

const projectListQueryOptions = (payload: ProjectListGetPayload) => {
  return queryOptions({
    queryKey: [PROJECTS_QUERY_KEY, payload],
    queryFn: () => fetchProjectList(payload),
  });
};

/**
 * Delete a Project
 * DELETE /projects/:uuid/
 */

const deleteProject = async (uuid: string) => {
  const res = await api.delete(PROJECTS_ENDPOINT + uuid + "/");
  return res.data;
};

const useDeleteProject = () => {
  return useMutation({
    mutationKey: [PROJECTS_MUTATION_KEY],
    mutationFn: (uuid: string) => deleteProject(uuid),
  });
};

export {
  projectGetQueryOptions,
  fetchProject,
  projectListQueryOptions,
  fetchProjectList,
  useDeleteProject,
  deleteProject,
  useCreateProject,
  createProject,
};
```

## Form Component

```typescript
// projects/Projects.form.tsx
import type { FormEvent } from "react";
import { z } from "zod";
import { createFormHook, createFormHookContexts } from "@tanstack/react-form";
import { Input } from "@/components/ui/input.tsx";
import { Field, Label, ErrorMessage } from "@/components/ui/fieldset.tsx";
import { Button } from "@/components/ui/button.tsx";
import type {
  ProjectCreatePayload,
  ProjectUpdatePayload,
  ProjectsFormProps,
} from "@/components/projects/Projects.types.ts";

const { fieldContext, formContext } = createFormHookContexts();

const { useAppForm } = createFormHook({
  fieldComponents: {
    Input,
  },
  formComponents: {
    Button,
  },
  fieldContext,
  formContext,
});

const ProjectsForm = <T extends ProjectCreatePayload | ProjectUpdatePayload>({
  formId,
  onSubmit,
  isLoading = false,
  defaultValues,
  hideActions = false,
}: ProjectsFormProps<T>) => {
  // Type guard function to check if we have a ProjectUpdatePayload
  const isUpdatePayload = (
    value: ProjectCreatePayload | ProjectUpdatePayload | undefined
  ): value is ProjectUpdatePayload => {
    return value !== undefined && "uuid" in value;
  };

  const form = useAppForm({
    defaultValues: {
      url: defaultValues?.url ?? "",
      type:
        (defaultValues?.type as "WORDPRESS" | "SHOPIFY" | "CUSTOM") ??
        "WORDPRESS",
    },
    onSubmit: async ({ value }) => {
      // Use the type guard to safely access uuid
      const payload = isUpdatePayload(defaultValues)
        ? ({ ...value, uuid: defaultValues.uuid } as T)
        : (value as T);

      onSubmit?.(payload);
    },
  });

  const handleSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    event.stopPropagation();
    form.handleSubmit();
  };

  return (
    <form
      id={formId}
      action="#"
      method="POST"
      className="grid w-full max-w-sm grid-cols-1 gap-8"
      onSubmit={handleSubmit}
    >
      <Field>
        <form.Field
          name="url"
          validators={{
            onChange: z.string().url("Invalid URL"),
            onBlur: z.string().url("Invalid URL"),
          }}
        >
          {(field) => {
            return (
              <>
                <Label htmlFor={field.name}>URL</Label>
                <Input
                  id={field.name}
                  name={field.name}
                  value={field.state.value}
                  type="url"
                  onChange={(e) => field.handleChange(e.target.value)}
                  invalid={field.state.meta.errors.length > 0}
                />
                {field.state.meta.errors.length > 0 && (
                  <ErrorMessage>
                    {field.state.meta.errors?.[0]?.message}
                  </ErrorMessage>
                )}
              </>
            );
          }}
        </form.Field>
      </Field>
      {!hideActions && (
        <form.Subscribe
          selector={(state) => [state.canSubmit, state.isSubmitting]}
          children={([_, isSubmitting]) => (
            <Button type="submit">
              {isSubmitting || isLoading ? "..." : "Submit"}
            </Button>
          )}
        />
      )}
    </form>
  );
};

export { ProjectsForm };
```

## Table Component

```typescript
// projects/Projects.table.tsx
import type { MouseEvent } from "react";
import { EllipsisHorizontalIcon } from "@heroicons/react/16/solid";
import { Badge } from "@/components/ui/badge";
import {
  Dropdown,
  DropdownButton,
  DropdownItem,
  DropdownMenu,
} from "@/components/ui/dropdown";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";

import type {
  Project,
  ProjectsTableComponent,
} from "@/components/projects/Projects.types.ts";
import { Text } from "@/components/ui/text.tsx";

const ProjectsTable: ProjectsTableComponent = ({ data, onView, onDelete }) => {
  const handleOnDelete = (project: Project) => {
    onDelete?.(project);
  };

  return (
    <div className="mt-6 ring-1 ring-zinc-950/10 dark:ring-white/10 px-3 sm:mx-0 sm:rounded-lg">
      <Table className="[--gutter:--spacing(6)] sm:[--gutter:--spacing(8)]">
        <TableHead>
          <TableRow>
            <TableHeader>URL</TableHeader>
            <TableHeader>Status</TableHeader>
            <TableHeader className="relative w-0">
              <span className="sr-only">Actions</span>
            </TableHeader>
          </TableRow>
        </TableHead>
        <TableBody>
          {data.map((project) => (
            <TableRow key={project.uuid}>
              <TableCell>
                <div className="flex items-center gap-4">
                  <div>
                    <div className="font-medium">
                      {project.url.replace("https://", "")}
                      <span className="inline-flex ml-2">
                        <Text>({project.uuid})</Text>
                      </span>
                    </div>
                  </div>
                </div>
              </TableCell>
              <TableCell>
                <Badge color="lime">Online</Badge>
              </TableCell>
              <TableCell>
                <div className="-mx-3 -my-1.5 sm:-mx-2.5">
                  <Dropdown>
                    <DropdownButton plain aria-label="More options">
                      <EllipsisHorizontalIcon />
                    </DropdownButton>
                    <DropdownMenu anchor="bottom end">
                      <DropdownItem
                        href="#"
                        onClick={(event: MouseEvent<HTMLButtonElement>) => {
                          event.preventDefault();
                          event.stopPropagation();
                          onView?.(project);
                        }}
                      >
                        View
                      </DropdownItem>
                      <DropdownItem
                        onClick={() => {
                          handleOnDelete(project);
                        }}
                      >
                        Delete
                      </DropdownItem>
                    </DropdownMenu>
                  </Dropdown>
                </div>
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </div>
  );
};

export { ProjectsTable };
```

## Page Component

```typescript
// projects/Projects.page.tsx
import { useState } from "react";
import { toast } from "react-hot-toast";
import { useSuspenseQuery } from "@tanstack/react-query";
import { getRouteApi } from "@tanstack/react-router";
import { PlusIcon } from "@heroicons/react/20/solid";
import {
  Project,
  ProjectCreatePayload,
  ProjectListGetPayload,
} from "@/components/projects/Projects.types.ts";
import {
  projectListQueryOptions,
  useDeleteProject,
  useCreateProject,
} from "@/components/projects/Projects.api.ts";
import { Heading } from "@/components/ui/heading.tsx";
import { Button } from "@/components/ui/button.tsx";
import { ProjectsEmpty } from "@/components/projects/Projects.empty.tsx";
import { ProjectsTable } from "@/components/projects/Projects.table.tsx";
import { Modal } from "@/components/common/modal/Modal.tsx";
import { ProjectsForm } from "@/components/projects/Projects.form.tsx";
import type {
  ImageListGetPayload,
  Image,
} from "@/components/images/Images.types.ts";
import { getDefaultSearchValues } from "@/utils.ts";
import { Breadcrumbs } from "@/components/ui/breadcrumbs.tsx";

const routeApi = getRouteApi("/_app/p/");

const ProjectsPage = () => {
  const [isDeleteModalOpen, setIsDeleteModalOpen] = useState(false);
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false);
  const [selectedProject, setSelectedProject] = useState<Project | null>(null);
  const search = routeApi.useSearch();
  const navigate = routeApi.useNavigate();

  // We need the default search parameters when we redirect to the project detail page
  // because we need to pass the search parameters to the image list query
  const imageListGetPayload: Omit<ImageListGetPayload, "project_uuid"> = {
    ...getDefaultSearchValues(),
    sort_by: "url" as keyof Image,
  };

  const projectListPayload: ProjectListGetPayload = {
    ...search,
  };

  const projectsQuery = useSuspenseQuery(
    projectListQueryOptions(projectListPayload)
  );

  /**
   * Create a Project
   * POST /projects/
   */

  // Mutation hook
  const createProject = useCreateProject();

  // Form callback
  const onCreate = (payload: ProjectCreatePayload) => {
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
  };

  /** End of Create a Project */

  /**
   * Delete a Project
   * DELETE /projects/:uuid/
   */

  // Mutation hook
  const deleteProject = useDeleteProject();

  // Table callback
  const onTableItemDelete = (project: Project) => {
    setSelectedProject(project);
    setIsDeleteModalOpen(true);
  };

  // Form callback
  const onDelete = () => {
    if (selectedProject) {
      let toastId = toast.loading("Deleting project...");
      deleteProject.mutate(selectedProject.uuid, {
        onSuccess: () => {
          projectsQuery.refetch();
          setIsDeleteModalOpen(false);
          setSelectedProject(null);
          toast.success("Project deleted successfully", { id: toastId });
        },
        onError: () => {
          toast.error("Unable to delete project", { id: toastId });
        },
      });
    }
  };

  /** End of Delete a Project */

  /**
   * Table callbacks
   */

  const onTableItemView = async (project: Project) => {
    await navigate({
      to: "/p/$puuid",
      params: { puuid: project.uuid },
      search: {
        ...imageListGetPayload,
      },
    });
  };

  /** End of Table callbacks */

  /**
   * Modal callbacks
   */

  const onClose = () => {
    setIsDeleteModalOpen(false);
    setSelectedProject(null);
  };

  /** End of Modal callbacks */

  let breadcrumbPages = [
    {
      name: "Projects",
      href: "/p",
      current: true,
    },
  ];

  return (
    <>
      {isDeleteModalOpen && (
        <Modal
          title={"Are you sure you want to delete this project?"}
          description={
            "All the images associated with the project will be deleted. It won't affect the website itself."
          }
          isOpen={isDeleteModalOpen}
          setIsOpen={setIsDeleteModalOpen}
          onConfirm={onDelete}
          onClose={onClose}
        />
      )}

      {isCreateModalOpen && (
        <Modal
          formId="create-project-form"
          title={"Create a new project"}
          description={
            "Please enter the URL of the project you want to create."
          }
          isOpen={isCreateModalOpen}
          setIsOpen={setIsCreateModalOpen}
          onClose={() => setIsCreateModalOpen(false)}
          isLoading={createProject.isPending}
        >
          <ProjectsForm
            hideActions
            formId="create-project-form"
            onSubmit={(values) => {
              // @ts-ignore
              onCreate(values);
            }}
            isLoading={false}
          />
        </Modal>
      )}
      <div className="flex py-4">
        <Breadcrumbs pages={breadcrumbPages} />
      </div>
      <div className="flex w-full flex-wrap items-end justify-between gap-4 border-b border-zinc-950/10 pb-6 dark:border-white/10">
        <Heading>Projects</Heading>
        <div className="flex gap-4">
          <Button onClick={() => setIsCreateModalOpen(true)}>
            <PlusIcon aria-hidden="true" className="-ml-0.5 mr-1.5 size-5" />
            New Project
          </Button>
        </div>
      </div>
      {projectsQuery.data.total === 0 ? (
        <ProjectsEmpty onCreate={() => setIsCreateModalOpen(true)} />
      ) : (
        <div>
          <ProjectsTable
            data={projectsQuery.data.items}
            onView={onTableItemView}
            onDelete={onTableItemDelete}
          />
        </div>
      )}
    </>
  );
};

export { ProjectsPage };
```

## Modal Component

```typescript
// components/common/modal/Modal.tsx
import { Fragment, ReactNode } from "react";
import { Dialog, Transition } from "@headlessui/react";
import { Button } from "@/components/ui/button.tsx";

interface ModalProps {
  title: string;
  description?: string;
  isOpen: boolean;
  setIsOpen: (isOpen: boolean) => void;
  onClose?: () => void;
  onConfirm?: () => void;
  children?: ReactNode;
  formId?: string;
  isLoading?: boolean;
}

const Modal = ({
  title,
  description,
  isOpen,
  setIsOpen,
  onClose,
  onConfirm,
  children,
  formId,
  isLoading = false,
}: ModalProps) => {
  const handleClose = () => {
    setIsOpen(false);
    onClose?.();
  };

  return (
    <Transition.Root show={isOpen} as={Fragment}>
      <Dialog as="div" className="relative z-10" onClose={handleClose}>
        <Transition.Child
          as={Fragment}
          enter="ease-out duration-300"
          enterFrom="opacity-0"
          enterTo="opacity-100"
          leave="ease-in duration-200"
          leaveFrom="opacity-100"
          leaveTo="opacity-0"
        >
          <div className="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" />
        </Transition.Child>

        <div className="fixed inset-0 z-10 w-screen overflow-y-auto">
          <div className="flex min-h-full items-end justify-center p-4 text-center sm:items-center sm:p-0">
            <Transition.Child
              as={Fragment}
              enter="ease-out duration-300"
              enterFrom="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
              enterTo="opacity-100 translate-y-0 sm:scale-100"
              leave="ease-in duration-200"
              leaveFrom="opacity-100 translate-y-0 sm:scale-100"
              leaveTo="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
            >
              <Dialog.Panel className="relative transform overflow-hidden rounded-lg bg-white px-4 pb-4 pt-5 text-left shadow-xl transition-all sm:my-8 sm:w-full sm:max-w-lg sm:p-6">
                <div>
                  <div className="mt-3 text-center sm:mt-5">
                    <Dialog.Title
                      as="h3"
                      className="text-base font-semibold leading-6 text-gray-900"
                    >
                      {title}
                    </Dialog.Title>
                    {description && (
                      <div className="mt-2">
                        <p className="text-sm text-gray-500">{description}</p>
                      </div>
                    )}
                    {children && <div className="mt-4">{children}</div>}
                  </div>
                </div>
                <div className="mt-5 sm:mt-6 sm:grid sm:grid-flow-row-dense sm:grid-cols-2 sm:gap-3">
                  {onConfirm && (
                    <Button
                      type={formId ? "submit" : "button"}
                      form={formId}
                      onClick={formId ? undefined : onConfirm}
                      className="inline-flex w-full justify-center sm:col-start-2"
                      disabled={isLoading}
                    >
                      {isLoading ? "..." : "Confirm"}
                    </Button>
                  )}
                  <Button
                    type="button"
                    onClick={handleClose}
                    className="mt-3 inline-flex w-full justify-center sm:col-start-1 sm:mt-0"
                    variant="secondary"
                  >
                    Cancel
                  </Button>
                </div>
              </Dialog.Panel>
            </Transition.Child>
          </div>
        </div>
      </Dialog>
    </Transition.Root>
  );
};

export { Modal };
```

## Empty State Component

```typescript
// projects/Projects.empty.tsx
import { PlusIcon } from "@heroicons/react/24/outline";
import { Button } from "@/components/ui/button.tsx";

interface ProjectsEmptyProps {
  onCreate: () => void;
}

const ProjectsEmpty = ({ onCreate }: ProjectsEmptyProps) => {
  return (
    <div className="text-center py-12">
      <svg
        className="mx-auto h-12 w-12 text-gray-400"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
        aria-hidden="true"
      >
        <path
          vectorEffect="non-scaling-stroke"
          strokeLinecap="round"
          strokeLinejoin="round"
          strokeWidth={2}
          d="M9 13h6m-3-3v6m-9 1V7a2 2 0 012-2h6l2 2h6a2 2 0 012 2v8a2 2 0 01-2 2H5a2 2 0 01-2-2z"
        />
      </svg>
      <h3 className="mt-2 text-sm font-semibold text-gray-900">No projects</h3>
      <p className="mt-1 text-sm text-gray-500">
        Get started by creating a new project.
      </p>
      <div className="mt-6">
        <Button onClick={onCreate}>
          <PlusIcon className="-ml-0.5 mr-1.5 h-5 w-5" aria-hidden="true" />
          New Project
        </Button>
      </div>
    </div>
  );
};

export { ProjectsEmpty };
```

## Project Structure

```
/src
  /components
    /[feature]
      [Feature].types.ts    # Type definitions
      [Feature].api.ts      # API integration
      [Feature].constants.ts # Constants
      [Feature].page.tsx    # Page component
      [Feature].form.tsx    # Form component
      [Feature].table.tsx   # Table component
      [Feature].detail.tsx  # Detail component
      [Feature].empty.tsx   # Empty state component
  /components
    /ui                     # Shared UI components
    /common                 # Shared common components
  /api.ts                   # Global API setup
  /constants.ts             # Global constants
  /types.ts                 # Global type definitions
  /utils.ts                 # Global utility functions
```

## Routing Conventions

### TanStack Router Integration

```typescript
// Basic route without data dependencies
import { createFileRoute } from "@tanstack/react-router";
import { AppPage } from "@/components/pages/AppPage.tsx";

export const Route = createFileRoute("/_app/")({
  component: AppPage,
});

// Data-dependent route
export const Route = createFileRoute("/_app/p/$puuid/")({
  validateSearch: (search?: Record<string, unknown>) => {
    return {
      ...getDefaultSearchValues<Image>(search),
      sort_by: "url" as keyof Image,
    };
  },
  loaderDeps: ({ search }) => {
    return search;
  },
  loader: async ({ context: { queryClient }, params, deps }) => {
    // Prefetch required data
    await queryClient.prefetchQuery(dataQueryOptions(payload));
  },
  component: FeatureDetail,
});
```

### Search Parameters

```typescript
const searchParams = {
  ...getDefaultSearchValues<EntityType>(search),
  sort_by: "name" as keyof EntityType,
};
```

### Data Loading

```typescript
const payload: EntityListGetPayload = {
  uuid: params.uuid,
  limit: deps.limit,
  offset: deps.offset,
  sort_by: deps.sort_by,
  sort_direction: deps.sort_direction,
  filter: deps.filter,
};

await queryClient.prefetchQuery(entityQueryOptions(payload));
```
