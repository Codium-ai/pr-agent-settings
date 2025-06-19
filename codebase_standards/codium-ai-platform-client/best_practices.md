# Code Best Practices

## Code Conventions

### Simplify "If" statement

**Good:**

```typescript
const isUserEligibleForDiscount =
  user.age >= 18 && user.hasValidMembership && !user.hasUsedDiscount;

if (isUserEligibleForDiscount) {
  applyDiscount();
}
```

**Bad:**

```typescript
if (user.age >= 18 && user.hasValidMembership && !user.hasUsedDiscount) {
  applyDiscount();
}
```

### Keep logic out of hooks

The motivation here is to keep hooks as small and simple as possible. Also, moving the logic out to small, testable function will make writing unit tests a much easier tasks (instead of needing to mock hooks, etc)

**Good:**

```typescript
// utils/url.ts
const extractPathSegments = (pathname: string): string[] => {
  return pathname.split('/').filter(Boolean);
};

// hooks/usePathSegments.ts
const usePathSegments = (): string[] => {
  const location = useLocation();
  return extractPathSegments(location.pathname);
};
```

**Bad:**

```typescript
const usePathSegments = (): string[] => {
  const location = useLocation();
  return location.pathname.split('/').filter(Boolean);
};
```

### Always add jsdoc to utils functions and hooks

**Good:**

```typescript
// utils/url.ts
/**
 * Extracts path segments from a URL pathname
 * @param pathname - The URL pathname to extract segments from
 * @returns Array of non-empty path segments
 */
const extractPathSegments = (pathname: string): string[] => {
  return pathname.split('/').filter(Boolean);
};

// hooks/usePathSegments.ts
/**
 * Hook that returns the current URL path segments
 * @returns Array of non-empty path segments from the current URL
 */
const usePathSegments = (): string[] => {
  const location = useLocation();
  return extractPathSegments(location.pathname);
};
```

**Bad:**

```typescript
const extractPathSegments = (pathname: string): string[] => {
  return pathname.split('/').filter(Boolean);
};

const usePathSegments = (): string[] => {
  const location = useLocation();
  return extractPathSegments(location.pathname);
};
```

### Use named export, no default export

**Good:**

```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;
export const subtract = (a: number, b: number): number => a - b;

// hooks/useCounter.ts
export const useCounter = (initialValue: number) => {
  const [count, setCount] = useState(initialValue);
  return { count, setCount };
};
```

**Bad:**

```typescript
// utils/math.ts
const add = (a: number, b: number): number => a + b;
const subtract = (a: number, b: number): number => a - b;

export default { add, subtract };

// hooks/useCounter.ts
const useCounter = (initialValue: number) => {
  const [count, setCount] = useState(initialValue);
  return { count, setCount };
};

export default useCounter;
```

### Always use barrel files

**Good:**

```typescript
// utils/math/add.ts
export const add = (a: number, b: number): number => a + b;

// utils/math/subtract.ts
export const subtract = (a: number, b: number): number => a - b;

// utils/math/index.ts
export * from './add';
export * from './subtract';

// Usage in other files
import { add, subtract } from '@/utils/math';
```

**Bad:**

```typescript
// utils/math/add.ts
export const add = (a: number, b: number): number => a + b;

// utils/math/subtract.ts
export const subtract = (a: number, b: number): number => a - b;

// Usage in other files
import { add } from '@/utils/math/add';
import { subtract } from '@/utils/math/subtract';
```

## Unit Tests

### Unit tests should be located right next to the code

**Good:**

```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;

// utils/math.spec.ts
import { add } from './math';

describe('add', () => {
  // Test code here
});
```

**Bad:**

```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;

// tests/utils/math.test.ts
import { add } from '../../../../utils/math';

describe('add', () => {
  // Test code here
});
```
