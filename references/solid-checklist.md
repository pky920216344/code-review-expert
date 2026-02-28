### Advanced Architecture and Design Rules

- **DRY** (Don’t Repeat Yourself): Avoid duplication by abstracting common patterns.
- **KISS** (Keep It Simple, Stupid): Simplicity is key, aim for simplicity in design.
- **Law of Demeter**: Promote loose coupling by limiting knowledge between different modules.
- **Conway's Law**: The structure of a system mirrors the communication structure of the organization.
- **Domain Driven Design**: Focus on the core domain and its logic to align software design and business needs.
- **Functional Core/Imperative Shell**: Keep the core of the application functional and pure, while managing side effects in the imperative layer.
- **Cyclomatic Complexity**: Aim for low complexity in your code to improve maintainability.
- **Deeply Nested Conditionals**: Reduce or eliminate nested structures to enhance readability.
- **Inappropriate Intimacy**: Classes that know too much about each other should be refactored to respect boundaries.
- **Primitive Obsession**: Avoid using primitive data types for complex data structures, use classes instead.  


**Last updated on: 2026-02-28 03:11:49**