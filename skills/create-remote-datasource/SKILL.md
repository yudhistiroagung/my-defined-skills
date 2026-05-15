---
name: create-remote-datasource
description: >
  Use this skill when creating a NEW remote datasource in the data layer.
  This sets up an Axios-backed remote datasource: datasource interface,
  remote datasource class, and DI container registration.
  Triggers: "create remote datasource", "add API datasource", "scaffold remote data", "new API source".
  Do NOT use for local datasources (use `create-local-datasource`) or
  repository implementation (use `create-repository-implementation`).
author: yudhistiroagung
---

## Overview
Creates a remote datasource that handles API calls via Axios.

The structure produced:
```
src/data/{data-name}/datasources/
├── {data-name}-datasource.ts             # shared datasource interface (create if not exists)
└── remote/
    └── {data-name}-remote-datasource.ts  # implementation class
```

---

## Step 1 — Create the datasource interface
Create `src/data/{data-name}/datasources/{data-name}-datasource.ts` if it does not exist yet.
This is the shared contract used by both local and remote implementations.

```ts
// src/data/todos/datasources/todo-datasource.ts
export interface TodoDataSource<T> {
  getTodos(): Promise<T[]>;
  createTodo(item: T): Promise<T>;
}
```

> Only add methods this feature actually needs.
> If this file already exists (e.g. local datasource was created first), skip this step.

---

## Step 2 — Create the remote datasource class
Create `src/data/{data-name}/datasources/remote/{data-name}-remote-datasource.ts`.

```ts
// src/data/todos/datasources/remote/todo-remote-datasource.ts
import { inject, singleton } from 'tsyringe';
import type { AxiosInstance } from 'axios';

import { AxiosCore } from '@/cores/axios';
import type { TodoDataSource } from '../todo-datasource';
import type { TodoDto } from '../../models/todo-dto';

@singleton()
export class TodoRemoteDatasource implements TodoDataSource<TodoDto> {
  static readonly TOKEN = 'TodoRemoteDatasource';

  constructor(
    @inject(AxiosCore.TOKEN) private readonly http: AxiosInstance
  ) {}

  async getTodos(): Promise<TodoDto[]> {
    const response = await this.http.get<TodoDto[]>('/todos');
    return response.data;
  }

  async createTodo(item: TodoDto): Promise<TodoDto> {
    const response = await this.http.post<TodoDto>('/todos', item);
    return response.data;
  }
}
```

> The remote datasource works with DTOs only — never domain models.
> Mapping from DTO to domain model is the repository's responsibility.

---

## Step 3 — Register in the DI container
Add the registration to `src/data/di/index.ts`:

```ts
// src/data/di/index.ts
import { container } from 'tsyringe';
import { TodoRemoteDatasource } from '../todos/datasources/remote/todo-remote-datasource';

container.register(TodoRemoteDatasource.TOKEN, TodoRemoteDatasource);
```

> Axios itself is registered once globally in `src/cores/axios` — do not re-register it here.

---

## What This Skill Does NOT Cover
- Local datasource (Dexie/IndexedDB) → use `create-local-datasource`
- Repository implementation that uses this datasource → use `create-repository-implementation`
- DTO-to-domain mapping → see `data-layer-scaffold` Step 5
- Domain model / repository interface → use `create-domain-layer`