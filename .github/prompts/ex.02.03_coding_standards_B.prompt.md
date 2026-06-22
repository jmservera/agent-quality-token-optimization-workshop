---
model: MAI-Code-1-Flash (copilot)
agent: Ask
---

Here are some coding standards:

# TypeScript Coding Standards

```typescript
import express, { Request, Response } from 'express';
import { addTask, getAllTasks, getNextId } from './store';
import { filterByMinPriority, topTasks } from './tasks';
import { Priority, Task } from './types';

export const router = express.Router();

// GET /tasks  -> all tasks, optionally filtered by ?minPriority=
router.get('/tasks', (req: Request, res: Response) => {
  const min = req.query.minPriority;
  if (min !== undefined) {
    return res.json(filterByMinPriority(Number(min) as Priority));
  }
  return res.json(getAllTasks());
});

// GET /tasks/top?limit=  -> highest priority tasks
router.get('/tasks/top', (req: Request, res: Response) => {
  const limit = Number(req.query.limit);
  return res.json(topTasks(limit));
});

// POST /tasks  -> create a task
// SECURITY/VALIDATION GAP (planted): the request body is trusted as-is.
// `title` and `priority` are never validated, so empty titles, missing
// fields, huge payloads, or out-of-range priorities are all accepted.
router.post('/tasks', (req: Request, res: Response) => {
  const body = req.body as { title: string; priority: Priority };
  const task: Task = {
    id: getNextId(),
    title: body.title,
    priority: body.priority,
    done: false,
    createdAt: new Date().toISOString(),
  };
  addTask(task);
  return res.status(201).json(task);
});

// GET /tasks/search?q=  -> search task titles
// SECURITY GAP (planted): builds a RegExp directly from user input.
// This allows ReDoS (catastrophic backtracking) from a crafted `q`.
router.get('/tasks/search', (req: Request, res: Response) => {
  const q = String(req.query.q ?? '');
  const pattern = new RegExp(q);
  const results = getAllTasks().filter((t) => pattern.test(t.title));
  return res.json(results);
});
```

1. Enable `strict` mode in `tsconfig.json` and never disable it per file.
2. Prefer `const` over `let`; never use `var`.
3. Avoid `any`; use `unknown` with narrowing or precise types instead.
4. Always declare explicit return types on exported functions and methods.
5. Use `interface` for object shapes and `type` for unions and intersections.
6. Name types and classes in `PascalCase`; variables and functions in `camelCase`.
7. Name constants that are truly fixed in `UPPER_SNAKE_CASE`.
8. Prefer named exports over default exports for discoverability.
9. Keep files focused: one primary responsibility per module.
10. Use `async`/`await` instead of raw `.then()` promise chains.
11. Always handle promise rejections; never leave a floating promise.
12. Validate and narrow external input at system boundaries.
13. Prefer immutable data: use `readonly` and `as const` where applicable.
14. Use optional chaining `?.` and nullish coalescing `??` for safe access.
15. Avoid non-null assertions `!`; narrow types explicitly instead.
16. Prefer `enum`-free unions of string literals for fixed value sets.
17. Keep functions small and pure; isolate side effects.
18. Use early returns to reduce nesting and improve readability.
19. Throw `Error` instances, never strings or plain objects.
20. Avoid magic numbers and strings; name them as constants.
21. Group and order imports: external, then internal, then relative.
22. Remove unused imports, variables, and dead code before committing.
23. Write self-documenting names; add comments only for non-obvious intent.
24. Use template literals instead of string concatenation.
25. Prefer array/object destructuring for clarity.
26. Type all function parameters; avoid implicit `any`.
27. Keep public APIs documented with concise JSDoc when non-trivial.
28. Run the linter and formatter (ESLint + Prettier) before every commit.
29. Cover business logic with unit tests; keep tests deterministic.
30. Fix all type errors; never ship with `@ts-ignore` left unexplained.
31. Keep modules small: prefer several focused files over one large file.
32. Export minimal surface area: only export what callers need.
33. Prefer composition over inheritance for reusable behaviors.
34. Use descriptive error messages with contextual information.
35. Avoid deep nesting; split complex logic into helper functions.
36. Use explicit casts sparingly; prefer type-safe transformations.
37. Prefer `Record<K, V>` for predictable object maps when appropriate.
38. Use `Readonly<T>` for public APIs that must not be mutated.
39. Favor pure functions in business logic; isolate I/O at boundaries.
40. Name boolean-returning functions with `is`/`has`/`can` prefixes.
41. Prefer small, focused interfaces; extend them explicitly when needed.
42. Use generics to express reusable abstractions with constraints.
43. Document type assumptions when using complex generics.
44. Keep side-effecting code in well-tested integration layers.
45. Avoid runtime type checks scattered throughout — centralize validation.
46. Use `tsconfig` paths for clear, maintainable import aliases.
47. Prefer `Map`/`Set` for collections with non-trivial lookup semantics.
48. Choose stable dependency versions; pin devDependencies for reproducibility.
49. Prefer `Promise.allSettled` when aggregating heterogeneous promises.
50. Use meaningful commit messages and link to issue trackers when relevant.
51. Keep tests small and single-concern; avoid brittle, timing-dependent tests.
52. Mock only external boundaries; prefer real instances for internal logic.
53. Use type guards to encapsulate runtime validation logic.
54. Avoid duplicating logic; extract shared utilities with clear ownership.
55. Keep configuration in typed, validated structures (`zod`, `io-ts`, etc.).
56. Prefer explicit feature flags over ad-hoc environment checks.
57. Use semantic versioning for public packages and document breaking changes.
58. Use small, focused pull requests to simplify reviews.
59. Prefer declarative APIs over imperative ones for clarity.
60. Use `never` to indicate impossible branches in exhaustive checks.
61. Prefer `Array.prototype` helpers (`map`, `filter`, `reduce`) for clarity.
62. Avoid heavy runtime libraries when lighter alternatives suffice.
63. Use bundler config to exclude server-only code from client builds.
64. Prefer readonly tuples for fixed-shape arrays with known lengths.
65. Use `Partial<T>`/`Required<T>` judiciously; prefer precise types.
66. Ensure sensitive data is never logged or committed to source control.
67. Run type-check in CI and block merges on type errors.
68. Prefer feature-oriented folders for large projects (domain-by-domain).
69. Keep migration scripts idempotent and tested against staging data.
70. Write small integration tests that exercise end-to-end behavior.
71. Use documented APIs for third-party libs; avoid private/undocumented hooks.
72. Keep performance-critical code measured and profile-guided.
73. Keep accessibility and internationalization considerations in mind.
74. Continuously improve type coverage and reduce `unknown`/`any` usage.
75. Periodically review and remove legacy code and deprecated APIs.

```typescript
import { filterByMinPriority, topTasks } from './tasks';

describe('topTasks', () => {
  it('returns the highest priority tasks first', () => {
    const top = topTasks(2);
    expect(top).toHaveLength(2);
    expect(top[0].priority).toBe(5);
    expect(top[1].priority).toBe(5);
  });

  it('never returns more than the requested limit', () => {
    expect(topTasks(1)).toHaveLength(1);
  });
});

describe('filterByMinPriority', () => {
  // This test passes: tasks strictly above the threshold are clearly included.
  it('includes tasks above the threshold', () => {
    const result = filterByMinPriority(2);
    expect(result.every((t) => t.priority >= 2)).toBe(true);
    expect(result.some((t) => t.priority === 5)).toBe(true);
  });

  // This test FAILS on purpose — it documents the planted off-by-one bug.
  // "minimum priority" should be INCLUSIVE, so priority === min must be returned.
  it('includes tasks exactly equal to the threshold (currently failing)', () => {
    const result = filterByMinPriority(2);
    const hasExactMatch = result.some((t) => t.priority === 2);
    expect(hasExactMatch).toBe(true);
  });
});

```

Every function must return a Result type, never throw.
Now, write a TypeScript function that divides two numbers, use minimal reasoning.