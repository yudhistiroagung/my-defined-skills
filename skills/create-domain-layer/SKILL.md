---
name: create-domain-layer
description: >
  Use this skill when creating a NEW domain layer for the project.
  This sets up the full domain layer structure: folder, Zod domain models, usecases, and repository interface (contract).
  Triggers: "create domain layer", "add new domain", "scaffold domain", "new domain model".
  Do NOT use for data layer, repository implementation, or UI layer.
author: yudhistiroagung
---

## Overview
Creates a complete domain layer for a new feature under `src/domain/{domain-name}/`.

The structure produced:
```
src/domain/{domain-name}/
├── usecases/                       # usecases
├── models/
│   └── {name}.ts                   # domain model (Zod)
└── {name}-repository.ts            # repository interface (contract only)
```

## Steps

**Step 1 — Create the folder**
Create `src/domain/{domain-name}/`.

**Step 2 — Define domain model** (in `models/`)
Use Zod Schema. The domain model is the **single source of truth** for this feature — it is not tied to local DB or remote API shape.

```ts
// src/domain/products/models/product.ts
import { z } from 'zod';

export const Product = z.object({
  id: z.string(),
  name: z.string(),
  price: z.number(),
});

export type Product = z.infer<typeof Product>;
```

**Step 3 — Create repository interface and TOKEN namespace**
Define the contract in `{name}-repository.ts`. This is an **interface only** — no implementation here.

```ts
// src/domain/products/product-repository.ts
import { Product } from './models/product';

export interface ProductRepository {
  getById(id: string): Promise<Product>;
  create(product: Product): Promise<Product>;
  update(product: Product): Promise<Product>;
  delete(id: string): Promise<void>;
}

export namespace ProductRepository {
  export const TOKEN = 'ProductRepository';
}
```

> Only include methods that this domain actually needs — don't copy the full CRUD list blindly.

**Step 4 — Create usecases**
Follow the linked skill: `create-domain-usecase`

## What This Skill Does NOT Cover
- Model-to-model mapping → use `data-layer-scaffold`
- Repository implementation → use `create-repository-implementation`
- Datasources → use `create-local-datasource` or `create-remote-datasource`