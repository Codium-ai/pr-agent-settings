<b>Pattern 1: Simplify "If" statements by extracting complex conditions into well-named variables for better readability and maintainability.</b>

Example code before:
```typescript
if (user.age >= 18 && user.hasValidMembership && !user.hasUsedDiscount) {
  applyDiscount();
}
```

Example code after:
```typescript
const isUserEligibleForDiscount =
  user.age >= 18 && user.hasValidMembership && !user.hasUsedDiscount;

if (isUserEligibleForDiscount) {
  applyDiscount();
}
```


<b>Pattern 2: Keep logic out of hooks by extracting it into separate, testable utility functions to improve code organization and testability.</b>

Example code before:
```typescript
const usePathSegments = (): string[] => {
  const location = useLocation();
  return location.pathname.split('/').filter(Boolean);
};
```

Example code after:
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


<b>Pattern 3: Always add JSDoc comments to utility functions and hooks to improve code documentation and developer experience.</b>

Example code before:
```typescript
const extractPathSegments = (pathname: string): string[] => {
  return pathname.split('/').filter(Boolean);
};

const usePathSegments = (): string[] => {
  const location = useLocation();
  return extractPathSegments(location.pathname);
};
```

Example code after:
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


<b>Pattern 4: Use named exports instead of default exports to improve code maintainability and enable better IDE refactoring support.</b>

Example code before:
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

Example code after:
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


<b>Pattern 5: Always use barrel files (index files) to centralize exports and simplify import statements across the codebase.</b>

Example code before:
```typescript
// utils/math/add.ts
export const add = (a: number, b: number): number => a + b;

// utils/math/subtract.ts
export const subtract = (a: number, b: number): number => a - b;

// Usage in other files
import { add } from '@/utils/math/add';
import { subtract } from '@/utils/math/subtract';
```

Example code after:
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


<b>Pattern 6: Unit tests should be co-located with the code they test to improve maintainability and make the relationship between code and tests clear.</b>

Example code before:
```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;

// tests/utils/math.test.ts
import { add } from '../../../../utils/math';

describe('add', () => {
  // Test code here
});
```

Example code after:
```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;

// utils/math.spec.ts
import { add } from './math';

describe('add', () => {
  // Test code here
});
```

