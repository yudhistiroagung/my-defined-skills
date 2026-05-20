---
name: react-usecase-hook
description: Generate React custom hooks that call domain use cases using TanStack Query. Use this skill whenever the user asks to create a custom hook, query hook, mutation hook, or any React hook that wraps a repository/use case call — especially when the project uses TanStack Query (@tanstack/react-query), dependency injection (DI), or a clean architecture pattern with domain models. Triggers include: "create a hook for [entity]", "make a useQuery hook", "wrap this repository method in a hook", "generate a custom hook for [use case]", or any request to expose a domain operation to React components.
---

# React Custom Hook with TanStack Query

Generates typed React custom hooks that bridge domain use cases / repositories to React components via TanStack Query.

## Project Conventions (infer from context if not stated)

| Convention | Default assumption |
|---|---|
| DI container | `import di from '@/di'` |
| Repository access | `di.repositories.<camelCaseName>` |
| Query options type | `import type { QueryOptions } from '@/cores/tanstack-query/tanstack-query'` |
| Domain models | `import type { ModelName } from '@/domain/<plural-entity>/models'` |
| Query keys | `SCREAMING_SNAKE_CASE` array, e.g. `['TODOS']` |

Always infer or ask about these if the user's project deviates from the defaults.

---

## Hook Types

### 1. Query Hook (read / GET)

Use `useQuery` for fetching data.

```typescript
import { useQuery } from '@tanstack/react-query';

import type { QueryOptions } from '@/cores/tanstack-query/tanstack-query';

import di from '@/di';
import type { Todo } from '@/domain/todos/models';

type TodoQueryOption = QueryOptions<Todo[]>;

export const useGetTodos = (options: TodoQueryOption = {}) => {
  const queryMethods = useQuery({
    queryKey: ['TODOS'],
    queryFn: () => di.repositories.todoRepository.getTodos(),
    ...options,
  });

  return queryMethods;
};
```

**Key rules for query hooks:**
- Hook name: `useGet<Entity>` or `useGet<Entity>By<Param>`
- `QueryOptions<ReturnType>` wraps TanStack's options for full override support
- Spread `...options` after defaults so callers can override `staleTime`, `enabled`, etc.
- Return the full `queryMethods` object (don't destructure inside the hook)

### 2. Query Hook with Parameters

When the use case requires an ID or filter:

```typescript
import { useQuery } from '@tanstack/react-query';

import type { QueryOptions } from '@/cores/tanstack-query/tanstack-query';

import di from '@/di';
import type { Todo } from '@/domain/todos/models';

type TodoByIdQueryOption = QueryOptions<Todo>;

export const useGetTodoById = (id: string, options: TodoByIdQueryOption = {}) => {
  const queryMethods = useQuery({
    queryKey: ['TODOS', id],
    queryFn: () => di.repositories.todoRepository.getTodoById(id),
    enabled: !!id,
    ...options,
  });

  return queryMethods;
};
```

**Key rules:**
- Include the param in `queryKey` to scope the cache entry
- Add `enabled: !!param` to prevent fetch with empty/undefined param
- Place required params before `options`

### 3. Mutation Hook (create / update / delete)

Use `useMutation` for write operations:

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';

import di from '@/di';
import type { CreateTodoPayload } from '@/domain/todos/models';

export const useCreateTodo = () => {
  const queryClient = useQueryClient();

  const mutationMethods = useMutation({
    mutationFn: (payload: CreateTodoPayload) =>
      di.repositories.todoRepository.createTodo(payload),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['TODOS'] });
    },
  });

  return mutationMethods;
};
```

**Key rules for mutation hooks:**
- Hook name: `useCreate<Entity>`, `useUpdate<Entity>`, `useDelete<Entity>`
- Always invalidate related query keys in `onSuccess`
- No `options` param by default unless the user requests it
- Import `useQueryClient` only when invalidation is needed

---

## Step-by-Step Generation Process

1. **Identify the operation type** — Is it a fetch (query) or write (mutation)?
2. **Determine the entity and method** — What domain model and repository method?
3. **Infer the return type** — Single item `Model`, list `Model[]`, or void
4. **Build the query key** — `['ENTITY_NAME']` for lists, `['ENTITY_NAME', id]` for single
5. **Check for parameters** — Does the use case need an ID, filter, or payload?
6. **Add invalidation** — Mutations should invalidate the relevant list query key
7. **Export the hook** — Named export only, no default export

---

## Naming Conventions

| Operation | Hook Name Pattern | Example |
|---|---|---|
| Fetch all | `useGet<Entities>` | `useGetTodos` |
| Fetch one | `useGet<Entity>ById` | `useGetTodoById` |
| Create | `useCreate<Entity>` | `useCreateTodo` |
| Update | `useUpdate<Entity>` | `useUpdateTodo` |
| Delete | `useDelete<Entity>` | `useDeleteTodo` |
| Custom use case | `use<VerbEntity>` | `useCompleteTodo` |

---

## Output Format

Always produce:
- A single `.ts` file (not `.tsx` — hooks don't return JSX)
- Full import block at the top (grouped: external → internal types → internal modules)
- Named export for the hook
- No inline comments unless logic is non-obvious
- TypeScript strict-compatible (no implicit `any`)

If the user provides multiple entities or operations, generate one hook per file and list all filenames clearly.