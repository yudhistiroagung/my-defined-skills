---
name: create-local-datasource
description: >
  Use this skill when creating a NEW local datasource in the data layer.
  This sets up a Dexie-backed local datasource: datasource interface, DB table config,
  local datasource class, and all DI container registrations.
  Triggers: "create local datasource", "add local DB source", "scaffold local data", "new dexie table".
  Do NOT use for remote datasources (use `create-remote-datasource`) or
  repository implementation (use `create-repository-implementation`).
author: yudhistiroagung
---

## Overview
Creates a complete local datasource backed by Dexie (IndexedDB).

The structure produced:
```
src/data/{data-name}/datasources/
├── {data-name}-datasource.ts         # shared datasource interface
└── local/
    ├── db/
    │   └── index.ts                  # Dexie table config + DI token
    └── {data-name}-local-datasource.ts  # implementation class
```

---

## Step 1 — Create the datasource interface
Create `src/data/{data-name}/datasources/{data-name}-datasource.ts` if it does not exist yet.
This is the shared contract used by both local and remote datasource implementations.

```ts
// src/data/todos/datasources/todo-datasource.ts
export interface TodoDataSource<T> {
  getTodos(): Promise<T[]>;
  setTodos(items: T[]): Promise<void>;
}
```

> Only add methods this feature actually needs.

---

## Step 2 — Create the Dexie table config
Create `src/data/{data-name}/datasources/local/db/index.ts`.

This file defines the table name, schema token, and TypeScript table type used by Dexie.

```ts
// src/data/todos/datasources/local/db/index.ts
import type { Table } from 'dexie';
import type { TodoEntity } from '../../../models/todo-entity';

export type TodoTable = Table<TodoEntity>;

export default {
  TOKEN: 'TodoLocalDBToken',   // used to inject the Dexie table
  TABLE_NAME: 'todos',         // must match AppDatabase table name (see Step 5)
};
```

---

## Step 3 — Create the local datasource class
Create `src/data/{data-name}/datasources/local/{data-name}-local-datasource.ts`.

```ts
// src/data/todos/datasources/local/todo-local-datasource.ts
import { inject, singleton } from 'tsyringe';

import type { TodoEntity } from '../../models/todo-entity';
import type { TodoDataSource } from '../todo-datasource';
import TodoLocalDb, { type TodoTable } from './db';

@singleton()
export class TodoLocalDatasource implements TodoDataSource<TodoEntity> {
  static readonly TOKEN = 'TodoLocalDatasource';

  constructor(
    @inject(TodoLocalDb.TOKEN) private readonly todos: TodoTable
  ) {}

  async getTodos(): Promise<TodoEntity[]> {
    return this.todos.toArray();
  }

  async setTodos(todos: TodoEntity[]): Promise<void> {
    return this.todos.bulkAdd(todos);
  }
}
```

---

## Step 4 — Register in the DI container
Add both registrations to `src/data/di/index.ts`.

**4a — Register the Dexie table:**
```ts
import { container } from 'tsyringe';
import { AppDatabase } from '@/cores/dexie/db-dexie';
import TodoLocalDb from '../todos/datasources/local/db';

container.register(
  ...AppDatabase.provideTable(TodoLocalDb.TOKEN, TodoLocalDb.TABLE_NAME),
);
```

**4b — Register the datasource implementation:**
```ts
import { TodoLocalDatasource } from '../todos/datasources/local/todo-local-datasource';

container.register(TodoLocalDatasource.TOKEN, TodoLocalDatasource);
```

> Both registrations go in the same `src/data/di/index.ts` file.

---

## Step 5 — Register the table in AppDatabase
Open `src/cores/dexie/db-dexie.ts` and add the new table:

1. Add the table name and indexed fields inside `initiate()`

```ts
// example — add alongside existing tables
class AppDatabase extends Dexie {

  initiate() {
    this.version(1).stores({
      todos: '++id, name',   // 1. table name + indexed fields
    });
  }
}
```

> `TABLE_NAME` in `db/index.ts` (Step 2) must exactly match the key used here.
> ONLY add new table name + indexed fields inside `initiate()`. Do NOT declare any table properties on the `AppDatabase` class since we provide them in the DI container.

```ts
// example — add alongside existing tables
class AppDatabase extends Dexie {

  initiate() {
    this.version(1).stores({
      todos: '++id, name',   // 1. table name + indexed fields
    });
  }
}
```

---

## What This Skill Does NOT Cover
- Remote datasource → use `create-remote-datasource`
- Repository implementation that uses this datasource → use `create-repository-implementation`
- Domain model / repository interface → use `create-domain-layer`