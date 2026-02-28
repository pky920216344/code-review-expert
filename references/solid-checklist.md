# SOLID Smell Prompts

## SRP (Single Responsibility)

- File owns unrelated concerns (e.g., HTTP + DB + domain rules in one file)
- Large class/module with low cohesion or multiple reasons to change
- Functions that orchestrate many unrelated steps
- God objects that know too much about the system
- **Ask**: "What is the single reason this module would change?"

## OCP (Open/Closed)

- Adding a new behavior requires editing many switch/if blocks
- Feature growth requires modifying core logic rather than extending
- No plugin/strategy/hook points for variation
- **Ask**: "Can I add a new variant without touching existing code?"

## LSP (Liskov Substitution)

- Subclass checks for concrete type or throws for base method
- Overridden methods weaken preconditions or strengthen postconditions
- Subclass ignores or no-ops parent behavior
- **Ask**: "Can I substitute any subclass without the caller knowing?"

## ISP (Interface Segregation)

- Interfaces with many methods, most unused by implementers
- Callers depend on broad interfaces for narrow needs
- Empty/stub implementations of interface methods
- **Ask**: "Do all implementers use all methods?"

## DIP (Dependency Inversion)

- High-level logic depends on concrete IO, storage, or network types
- Hard-coded implementations instead of abstractions or injection
- Import chains that couple business logic to infrastructure
- **Ask**: "Can I swap the implementation without changing business logic?"

---

## Domain & Architecture Smells (Modern/DDD)

- **Domain Leakage**: Using DTOs, HTTP requests, or DB entities directly in core business logic.
- **Missing Anti-Corruption Layer (ACL)**: Third-party API structures bleeding into internal models.
- **Anemic Models**: Objects that are just bags of getters/setters while Services contain all the logic.
- **Shared Mutable State**: Multiple concurrent flows modifying the same object/memory.
- **Async Contagion**: Unnecessary async functions that force the entire call chain to become asynchronous.

---

## Common Code Smells (Beyond SOLID)

| Smell | Signs |
|-------|-------|
| **Long method** | Function > 30 lines, multiple levels of nesting |
| **Feature envy** | Method uses more data from another class than its own |
| **Data clumps** | Same group of parameters passed together repeatedly |
| **Primitive obsession** | Using strings/numbers instead of domain types |
| **Shotgun surgery** | One change requires edits across many files |
| **Divergent change** | One file changes for many unrelated reasons |
| **Dead code** | Unreachable or never-called code |
| **Speculative generality** | Abstractions for hypothetical future needs |
| **Magic numbers/strings** | Hardcoded values without named constants |
| **Flag Arguments** | Passing boolean `true/false` to control internal function flow |
| **Temporal Coupling** | `callA()` must be called before `callB()` but compiler doesn't enforce it |
| **Hidden State** | Functions relying on global/env variables not passed as arguments |
| **Mocking Obsession** | Tests have excessive mocks, testing "how" instead of "what" |

---

## Refactor Heuristics

1. **Split by responsibility, not by size** - A small file can still violate SRP
2. **Introduce abstraction only when needed** - Wait for the second use case
3. **Keep refactors incremental** - Isolate behavior before moving
4. **Preserve behavior first** - Add tests before restructuring
5. **Name things by intent** - If naming is hard, the abstraction might be wrong
6. **Prefer composition over inheritance** - Inheritance creates tight coupling
7. **Make illegal states unrepresentable** - Use types to enforce invariants
8. **Push side-effects to the edges** - Keep core business logic as pure functions; handle I/O at the boundaries
9. **Default to Immutability** - Return new data structures instead of modifying inputs in-place
10. **Avoid Boolean Blindness** - Use Enums or specific types instead of boolean parameters for clarity