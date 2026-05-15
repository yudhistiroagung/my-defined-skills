---
name: create-react-presenter-hook
description: >
  Use this skill when creating a React presenter hook for a page or screen component.
  Every page has exactly one dedicated hook named {page-name}.hook.ts that owns
  all logic, state, and handlers — the page component only renders.
  Triggers: "create presenter hook", "add hook for page", "create page hook",
  "scaffold presenter", "add hook for screen", "create {page-name}.hook.ts".
  Do NOT use for shared/reusable hooks (use plain custom hooks in
  `presentation/hooks/`), or for hooks that are not tied to a specific page.
---

## Overview

Each page has exactly **one** presenter hook. It owns all logic for that page.
The page component imports it and renders — nothing else.

```
src/presentation/routes/{page-name}/
├── index.tsx               ← renders only, no logic
└── {page-name}.hook.ts     ← all state, handlers, effects
```

---

## Rules

- **One hook per page.** Never share a presenter hook between pages.
- **One effect per concern.** Each `useEffect` does exactly one thing.
- **No logic in the page component.** All handlers, derived state, and effects live in the hook.
- **Return only what the page needs.** Don't expose internal state that the page doesn't use.
- **No JSX in the hook.** The hook returns data and callbacks only.

---

## Step 1 — Create the hook file

Create `src/presentation/routes/{page-name}/{page-name}.hook.ts`.

The hook name must match the page: page `menu-1` → hook `useMenu1`.

---

## Step 2 — Follow the internal structure

Always declare in this order, with section comments:

```ts
// src/presentation/routes/{page-name}/{page-name}.hook.ts
import { useCallback, useEffect, useMemo, useState } from 'react';

export const use{PageName} = () => {

  /**
   * External library hooks (navigation, i18n, router, etc.)
   */


  /**
   * Custom hooks (data fetching, shared logic)
   */


  /**
   * State declarations
   */


  /**
   * Derived state (useMemo)
   */


  /**
   * Handlers (useCallback)
   */


  /**
   * Effects
   * 1 effect = 1 concern only
   */


  return {
    // expose only what the page component needs
  };
};
```

> Keep this order consistent across all presenter hooks — it makes every hook
> predictable to read regardless of which page it belongs to.

---

## Step 3 — Full example

```ts
// src/presentation/routes/menu-1/menu-1.hook.ts
import { useCallback, useEffect, useMemo, useState } from 'react';
import { useRouter } from '@tanstack/react-router';

import { useGetTodos } from '@/presentation/hooks/use-get-todos';

export const useMenu1 = () => {

  /**
   * External library hooks
   */
  const { navigate } = useRouter();

  /**
   * Custom hooks
   */
  const { data: todos } = useGetTodos();

  /**
   * State declarations
   */
  const [count, setCount] = useState(0);

  /**
   * Derived state
   */
  const mappedCount = useMemo(() => `This is ${count}`, [count]);

  /**
   * Handlers
   */
  const increaseCount = useCallback(() => {
    setCount((prev) => prev + 1);
  }, []);

  /**
   * Effects
   */
  useEffect(() => {
    console.log('TODOS', todos);
  }, [todos]);

  return {
    count,
    mappedCount,
    increaseCount,
    navigate,
  };
};
```

---

## Step 4 — Wire to the page component

The page component imports the hook and destructures what it needs. No logic, no state, no handlers inside the component.

```tsx
// src/presentation/routes/menu-1/menu-1.tsx
import { useMenu1 } from './menu-1.hook';

const Menu1Page = () => {
  const { count, mappedCount, increaseCount } = useMenu1();

  return (
    <div>
      <p>{mappedCount}</p>
      <button onClick={increaseCount}>Count: {count}</button>
    </div>
  );
};

export default Menu1Page;
```

> If you find yourself writing `useState`, `useEffect`, or handler functions
> directly in the page component — move them to the hook.

---

## Return value rules

Only return values the page component actually renders or calls:

| ✅ Return | ❌ Don't return |
|-----------|----------------|
| State used in JSX | Internal intermediate state |
| Derived values rendered in JSX | Raw data before transformation |
| Handlers attached to elements | Setters (expose handlers instead) |
| Navigation/router refs the page needs | Hooks the page doesn't directly use |

---

## What This Skill Does NOT Cover

- Shared reusable hooks (e.g. `useGetTodos`) → create in `src/presentation/hooks/`
- Data fetching hooks (TanStack Query wrappers) → use the data layer and query hooks
- Sub-component hooks → sub-components get their own hook only if they have significant logic; otherwise props suffice