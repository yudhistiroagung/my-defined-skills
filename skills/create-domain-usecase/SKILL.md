---
name: create-domain-usecase
description: >
  Use this skill when creating a NEW use case class in the domain layer.
  Each use case represents exactly one business operation, implemented as a
  class that extends BaseUseCase with typed input and output.
  Triggers: "create use case", "add use case", "new use case", "create domain logic",
  "add business logic", "scaffold use case", "create {action} use case".
  Do NOT use for repository interfaces (use `create-domain-layer`), repository
  implementation (use `create-repository-implementation`), or presentation hooks
  (use `create-presenter-hook`).
---

## Overview

Each use case is a single-responsibility class that executes one business operation.
It lives in the domain layer and depends only on repository interfaces — never on
data layer implementations directly.

The file produced:
```
src/domain/{domain-name}/usecases/
└── {action}-{domain-name}.ts           # e.g. get-product-by-id.ts
└── {action}-{domain-name}.test.ts      # e.g. get-product-by-id.test.ts
```

---

## Rules

- **One use case = one business operation.** Never combine two operations in one class.
- **Domain layer only.** Use cases depend on repository interfaces, not implementations.
- **Always typed.** `BaseUseCase<I, O>` must have explicit input and output types — never `any`.
- **Input as object.** Always use a named object `{ field }` as input, even for a single field. This makes future extension non-breaking.
- **Register in DI.** Every use case must be registered in `src/di/index.ts`.

---

## Step 1 — Name the use case file

Use the pattern `{verb}-{domain-name}.ts`, named after what it does:

| Operation | File name |
|-----------|-----------|
| Fetch one by ID | `get-product-by-id.ts` |
| Fetch all | `get-all-products.ts` |
| Create | `create-product.ts` |
| Update | `update-product.ts` |
| Delete | `delete-product.ts` |
| Business-specific | `calculate-product-discount.ts` |

Create the file at `src/domain/{domain-name}/usecases/{action}-{domain-name}.ts`.

---

## Step 2 — Implement the use case class

```ts
// src/domain/products/usecases/get-product-by-id.ts
import { singleton, inject } from 'tsyringe';

import type { BaseUseCase } from '@/core/usecase/base-usecase';
import type { Product } from '../models/product';
import type { ProductRepository } from '../product-repository';

type Input = {
  id: string;
};

type Output = Product;

@singleton()
export class GetProductById implements BaseUseCase<Input, Output> {
  static readonly TOKEN = 'GetProductById';

  constructor(
    @inject(ProductRepository.TOKEN)
    private readonly productRepository: ProductRepository,
  ) {}

  async run({ id }: Input): Promise<Output> {
    return this.productRepository.getProductById(id);
  }
}
```

> Define `Input` and `Output` as named types above the class — never inline them
> in the `implements` clause. This keeps the class signature readable.

---

## Step 3 — Register in the DI container

Add the registration to `src/domain/di/index.ts`:

```ts
import { container } from 'tsyringe';

import { GetProductsUsecase } from '../todos/usecases/use-get-products';

export default {
  getProductsUsecase: container.resolve(GetProductsUsecase),
};

```
---

## Step 5 — create unit test for the use case class

```ts
import { GetProductById } from '../usecases/get-product-by-id';

describe('GetProductById', () => {
  it('should return the product by ID', async () => {
    const usecase = new GetProductById(new MockProductRepositoryImpl());

    const product = await usecase.run({ id: '123' });

    expect(product).toBeDefined();
    expect(product.id).toBe('123');
  });
});
```
---

## Common patterns

### No input needed (fetch all)

```ts
type Input = void;
type Output = Product[];

@singleton()
export class GetAllProducts implements BaseUseCase<Input, Output> {
  static readonly TOKEN = 'GetAllProducts';

  constructor(
    @inject(ProductRepository.TOKEN)
    private readonly productRepository: ProductRepository,
  ) {}

  async run(): Promise<Output> {
    return this.productRepository.getAllProducts();
  }
}
```

### Multiple repository dependencies

```ts
@singleton()
export class CreateOrder implements BaseUseCase<Input, Output> {
  static readonly TOKEN = 'CreateOrder';

  constructor(
    @inject(OrderRepository.TOKEN)
    private readonly orderRepository: OrderRepository,

    @inject(ProductRepository.TOKEN)
    private readonly productRepository: ProductRepository,
  ) {}

  async run({ productId, quantity }: Input): Promise<Output> {
    const product = await this.productRepository.getProductById(productId);
    return this.orderRepository.createOrder({ product, quantity });
  }
}
```

> Only inject repositories the use case actually needs.

---

## What This Skill Does NOT Cover

- Repository interface definition → use `create-domain-layer`
- Repository implementation → use `create-repository-implementation`
- Calling the use case from UI → use `create-presenter-hook`
- Domain models (Zod schemas) → see `create-domain-layer`