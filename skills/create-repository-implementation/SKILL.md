---
name: create-repository-implementation
description: >
  Use this skill when creating a NEW repository implementation class in the data layer and its unit test file.
  This wires the domain repository interface to its actual datasource(s) using dependency injection.
  Triggers: "implement repository", "create repository impl", "wire datasource to repository".
  Do NOT use for defining the repository interface (use `create-domain-layer`) or
  creating datasources (use `create-local-datasource` or `create-remote-datasource`).
author: yudhistiroagung
---

## Overview
Creates the concrete class that implements the domain repository interface,
injects datasource(s), create unit test file, and then registers itself in the DI container.

The file produced:
```
src/data/{data-name}/
└── {data-name}-repository-impl.ts
└── {data-name}-repository-impl.test.ts
```

## Steps

**Step 1 — Create the implementation class**
Create `src/data/{data-name}/{data-name}-repository-impl.ts`.

- Decorate with `@singleton()`
- Implement the domain interface from `src/domain/{data-name}/{data-name}-repository.ts`
- Inject datasources via constructor using their `.TOKEN`, 
- create static field for repository TOKEN, e.g. `ProductRepositoryImpl.TOKEN = {data-name}Repository.TOKEN`

```ts
// src/data/products/product-repository-impl.ts
import { singleton, inject } from 'tsyringe';
import { ProductRepository } from '@/domain/products/product-repository';
import { ProductDataSource } from './datasources/product-datasource';
import { ProductLocalDataSource } from './datasources/product-local-datasource';
import { ProductRemoteDataSource } from './datasources/product-remote-datasource';
import { ProductEntity } from './models/product-entity';
import { ProductDto } from './models/product-dto';

@singleton()
export class ProductRepositoryImpl implements ProductRepository {
  static readonly TOKEN = ProductRepository.TOKEN;

  constructor(
    @inject(ProductLocalDataSource.TOKEN)
    private readonly localDataSource: ProductDataSource<ProductEntity>,

    @inject(ProductRemoteDataSource.TOKEN)
    private readonly remoteDataSource: ProductDataSource<ProductDto>,
  ) {}

  async create(product: Product): Promise<Product> {
    // call datasource, then map result to domain model
  }
}
```

**Step 2 — Create the unit test file**
Create `src/data/{data-name}/{data-name}-repository-impl.test.ts`. and implement the test cases.

> Inject only the datasources this repository actually uses.
> Not all repositories need both local and remote.

**Step 2 — Register and resolve in the DI container**
Add the binding in `src/data/di/index.ts`:

```ts
/**
 * Register Repositories
 */
container.register(ProductRepositoryImpl.TOKEN, ProductRepositoryImpl);

export default {
  // ... other repositories
  productRepository: container.resolve(ProductRepositoryImpl),
};
```

## What This Skill Does NOT Cover
- Repository interface definition → use `create-domain-layer`
- Datasource implementation → use `create-local-datasource` or `create-remote-datasource`
- Mapper functions → see `data-layer-scaffold` Step 5