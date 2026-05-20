---
name: data-layer-scaffold
description: >
  Use this skill when creating a NEW data layer for the project.
  This sets up the full data layer structure: folder, Zod models (entity + dto),
  datasources, repository implementation, and mapper functions.
  Triggers: "create data layer", "add new data", "scaffold data", "new repository".
  Do NOT use for modifying existing data layers or creating UI/domain layers.
author: yudhistiroagung
---

## Overview
Creates a complete data layer for a new feature under `src/data/{data-name}/`.

The structure produced:
```
src/
├── data/
|   └── products/
|       ├── datasources/
|       ├── mappers/
|       |   └── product-mapper.ts      
|       ├── models/
|       |   └── product-entity.ts      
|       |   └── product-dto.ts      
|       └── product-repository-impl.ts   
```

## Steps

**Step 1 — Create the folder**
Create `src/data/{data-name}/`.

**Step 2 — Define models** (in `models/`)
Use Zod Schema for all models. Two types:
- `{name}-entity.ts` — local database model
- `{name}-dto.ts` — remote API model

Both may not always be needed; create only what applies.

Example:
```ts
import { z } from 'zod';

export const ProductEntity = z.object({
  id: z.string(),
  name: z.string(),
  unitCost: z.number(),
});

export type ProductEntity = z.infer<typeof ProductEntity>;
```

**Step 3 — Create datasources**
At least one datasource is required. Follow the appropriate linked skill:
- Local → use skill `create-local-datasource`
- Remote → use skill `create-remote-datasource`

**Step 4 — Create repository implementation**
Follow the linked skill: `create-repository-implementation`

**Step 5 — Create mapper functions**
In `mapper/{name}-mapper.ts`, write functions to convert:
- entity ↔ domain model
- dto ↔ domain model
- must use `parse` method from Zod to validate the model.

Example:
```ts
export const productEntityDomain = (entity: ProductEntity): Product => {
  const mapped = { ... } // mapping here
  return Product.parse(mapped); // validate and return domain model
}
export const productDtoDomain = (entity: ProductEntity): Product => {
  const mapped = { ... } // mapping here
  return ProductDto.parse(mapped); // validate and return dto model
}
```

**Step 6 - Create unit test for mapper functions**
Create `src/data/{data-name}/mappers/{name}-mapper.test.ts`. and implement the test cases.