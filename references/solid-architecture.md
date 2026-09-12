# SOLID Architecture — Default Principles and Pluggability

Generic baseline for the code-writing and code-reviewing phases — [implement](../phases/implement.md), [review](../phases/review.md), [`audit code`](../phases/audit.md). Follow the SOLID principles unless project rules (`.dev_flow/rules/`, typically `architecture.md`) explicitly define alternative architectural conventions; project rules always take precedence.

## SOLID Principles

- **SRP (Single Responsibility):** Separate classes and functions by their logical concern. For example, database persistence logic must be separate from data processing logic.
- **OCP (Open/Closed):** Use subclasses or the Strategy pattern instead of endless if/else or switch statements when adding new behavior.
- **LSP (Liskov Substitution):** Ensure any subclass can replace its base class without breaking the program. Never throw "Not Implemented" errors from subclass methods.
- **ISP (Interface Segregation):** Create narrow, specialized interfaces. Do not force a class to implement methods it does not need.
- **DIP (Dependency Inversion):** Always depend on abstractions (interfaces), not concrete implementations. Use Dependency Injection.

## Pluggability Principles

Every module or functional block should be designed as an independent component:

- **Replaceability:** Use interfaces/abstractions so that any implementation can be swapped without changing the core system code.
- **Deactivation:** Provide the ability to fully deactivate a module (e.g., via configuration or a DI container) while the system continues to operate stably without that functionality.
- **No hard dependencies:** Avoid direct imports of concrete classes from one module into another. Use Events or Mediator patterns for cross-module communication.
