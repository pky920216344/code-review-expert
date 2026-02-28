# Code Quality Checklist

## Error Handling

### Anti-patterns to Flag

- **Swallowed exceptions**: Empty catch blocks or catch with only logging
  ```javascript
  try { ... } catch (e) { }  // Silent failure
  try { ... } catch (e) { console.log(e) }  // Log and forget
  ```
- **Overly broad catch**: Catching `Exception`/`Error` base class instead of specific types
- **Error information leakage**: Stack traces or internal details exposed to users
- **Missing error handling**: No try-catch around fallible operations (I/O, network, parsing)
- **Async error handling**: Unhandled promise rejections, missing `.catch()`, no error boundary
- **Catching without Re-throwing Context**: Catching an error and throwing a new one without preserving the original stack trace/cause
  ```javascript
  // Bad: original cause is lost
  try { ... } catch (e) { throw new Error('Operation failed') }
  // Good: original cause is preserved
  try { ... } catch (e) { throw new Error('Operation failed', { cause: e }) }
  ```
- **Error Codes as Strings/Magic Numbers**: Using hardcoded strings or numbers instead of strongly typed Error Classes or Enums
  ```javascript
  // Bad: magic string/number
  throw new Error('ERR_NOT_FOUND')
  if (error.code === 404) { ... }
  // Good: typed error class or enum
  throw new NotFoundError('Resource not found')
  if (error instanceof NotFoundError) { ... }
  ```

### Best Practices to Check

- [ ] Errors are caught at appropriate boundaries
- [ ] Error messages are user-friendly (no internal details exposed)
- [ ] Errors are logged with sufficient context for debugging
- [ ] Async errors are properly propagated or handled
- [ ] Fallback behavior is defined for recoverable errors
- [ ] Critical errors trigger alerts/monitoring
- [ ] External API errors are mapped to internal domain errors before propagating

### Questions to Ask
- "What happens when this operation fails?"
- "Will the caller know something went wrong?"
- "Is there enough context to debug this error?"

---

## Performance & Caching

### CPU-Intensive Operations

- **Expensive operations in hot paths**: Regex compilation, JSON parsing, crypto in loops
- **Blocking main thread**: Sync I/O, heavy computation without worker/async
- **Unnecessary recomputation**: Same calculation done multiple times
- **Missing memoization**: Pure functions called repeatedly with same inputs

### Frontend/Client Performance

- **Bundle Size Bloat**: Importing entire libraries when only a single function is needed
  ```javascript
  // Bad: imports entire lodash bundle
  import _ from 'lodash'
  // Good: imports only what is needed
  import { debounce } from 'lodash'
  ```
- **Unnecessary Re-renders**: Missing React `useMemo`/`useCallback` or equivalent in other frameworks in hot paths
- **Missing Pagination/Virtualization**: Rendering massive DOM lists without virtual scrolling

### Database & I/O

- **N+1 queries**: Loop that makes a query per item instead of batch
  ```javascript
  // Bad: N+1
  for (const id of ids) {
    const user = await db.query(`SELECT * FROM users WHERE id = ?`, id)
  }
  // Good: Batch
  const users = await db.query(`SELECT * FROM users WHERE id IN (?)`, ids)
  ```
- **Missing indexes**: Queries on unindexed columns
- **Over-fetching**: SELECT * when only few columns needed
- **No pagination**: Loading entire dataset into memory
- **Missing connection pooling**: Opening a new DB connection per request instead of using a pool

### Caching Issues

- **Missing cache for expensive operations**: Repeated API calls, DB queries, computations
- **Cache without TTL**: Stale data served indefinitely
- **Cache without invalidation strategy**: Data updated but cache not cleared
- **Cache key collisions**: Insufficient key uniqueness
- **Caching user-specific data globally**: Security/privacy issue

### Memory

- **Unbounded collections**: Arrays/maps that grow without limit
- **Large object retention**: Holding references preventing GC
- **String concatenation in loops**: Use StringBuilder/join instead
- **Loading large files entirely**: Use streaming instead

### Questions to Ask
- "What's the time complexity of this operation?"
- "How does this behave with 10x/100x data?"
- "Is this result cacheable? Should it be?"
- "Can this be batched instead of one-by-one?"

---

## Boundary Conditions

### Null/Undefined Handling

- **Missing null checks**: Accessing properties on potentially null objects
- **Truthy/falsy confusion**: `if (value)` when `0` or `""` are valid
- **Optional chaining overuse**: `a?.b?.c?.d` hiding structural issues
- **Null vs undefined inconsistency**: Mixed usage without clear convention

### Empty Collections

- **Empty array not handled**: Code assumes array has items
- **Empty object edge case**: `for...in` or `Object.keys` on empty object
- **First/last element access**: `arr[0]` or `arr[arr.length-1]` without length check

### Numeric Boundaries

- **Division by zero**: Missing check before division
- **Integer overflow**: Large numbers exceeding safe integer range
- **Floating point comparison**: Using `===` instead of epsilon comparison
- **Negative values**: Index or count that shouldn't be negative
- **Off-by-one errors**: Loop bounds, array slicing, pagination

### String Boundaries

- **Empty string**: Not handled as edge case
- **Whitespace-only string**: Passes truthy check but is effectively empty
- **Very long strings**: No length limits causing memory/display issues
- **Unicode edge cases**: Emoji, RTL text, combining characters

### Time & Date Boundaries

- **Timezone Ignorance**: Storing/comparing dates without normalizing to UTC
- **Leap Year/Leap Second Bugs**: Hardcoding 365 days or 24 hours assumptions
- **Daylight Saving Time**: Scheduled tasks or diff calculations that break during DST transitions

### Common Patterns to Flag

```javascript
// Dangerous: no null check
const name = user.profile.name

// Dangerous: array access without check
const first = items[0]

// Dangerous: division without check
const avg = total / count

// Dangerous: truthy check excludes valid values
if (value) { ... }  // fails for 0, "", false
```

### Questions to Ask
- "What if this is null/undefined?"
- "What if this collection is empty?"
- "What's the valid range for this number?"
- "What happens at the boundaries (0, -1, MAX_INT)?"

---

## Testing Quality

### Anti-patterns to Flag

- **Flaky Tests**: Tests that rely on `setTimeout`, exact timestamps, or external network calls
  ```javascript
  // Bad: timing-dependent, will fail under load
  await new Promise(resolve => setTimeout(resolve, 500))
  expect(result).toBe('done')
  ```
- **Missing Assertions**: Tests that execute code but don't actually `assert`/`expect` anything meaningful
  ```javascript
  // Bad: no assertion, test always passes
  it('should process items', () => {
    processItems([1, 2, 3])
  })
  ```
- **Implementation Coupling**: Tests that test private methods or internal state rather than public behavior (makes refactoring hard)
- **Low Branch Coverage**: Tests only cover the "happy path" and ignore the error paths or boundary conditions

### Questions to Ask
- "Does this test fail for the right reason when the code is broken?"
- "Would this test still pass if the feature were completely removed?"
- "Does this test cover the error path or only the success path?"
- "Could this test fail intermittently due to timing or external dependencies?"
