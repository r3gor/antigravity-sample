---
description: Scans the current code or recent changes to identify anti-patterns, SOLID violations, and opportunities for applying design patterns.
---

# Architectural & Pattern Review (/architecture-review)

This workflow acts as an automated Principal Engineer reviewing the codebase. It analyzes the specific files or recent changes passed by the user to ensure alignment with SOLID principles, framework best practices, and clean code standards.

## Instructions for the Agent:

When the user invokes this workflow (e.g., `/architecture-review src/domain/` or `/architecture-review` for the current context), strictly follow these steps:

1.  **Analyze the Target Code:**
    *   Examine the files, directories, or code diffs currently in context or provided by the user.
    *   Identify the programming language and the framework being used (e.g., TypeScript with NestJS, Node.js with Express).
    *   Review the code structure, class/function responsibilities, and dependency flow.

2.  **Evaluate Against SOLID Principles:**
    *   **Single Responsibility (SRP):** Are classes, functions, or modules doing too much?
    *   **Open/Closed (OCP):** Is the code open for extension but closed for modification?
    *   **Liskov Substitution (LSP):** Do subclasses or implementations break the expected behavior of their interfaces?
    *   **Interface Segregation (ISP):** Are interfaces too bloated, forcing clients to implement unused methods?
    *   **Dependency Inversion (DIP):** Does high-level policy depend on low-level details, or are both depending on abstractions (essential for Hexagonal Architecture)?

3.  **Detect Anti-Patterns and Code Smells:**
    *   Look for common code smells (e.g., God objects, magic numbers, deeply nested callbacks/ifs, implicit dependencies, tightly coupled layers).
    *   Flag any violations of the chosen framework's standard conventions or language idioms.

4.  **Identify Pattern Opportunities:**
    *   Suggest standard design patterns (e.g., Factory, Strategy, Observer, Repository, Adapter) where their application would genuinely improve maintainability or extensibility.
    *   Avoid suggesting patterns that would lead to over-engineering simple solutions.

5.  **Generate the Review Report:**
    *   Present your findings to the user using clear Markdown.
    *   Structure the report as follows:
        *   **🔴 Critical Violations / Anti-patterns:** Issues that actively harm maintainability and must be fixed.
        *   **🟡 Pattern & SOLID Opportunities:** Suggestions to refactor using a better design pattern or to strictly align with SOLID.
        *   **🟢 Clean Code Commendations:** Practices that were done exceptionally well.
    *   **Crucial:** For every issue found, **provide a short, concrete code snippet** showing the current state versus the proposed refactored solution.
    *   Finally, ask the user if they would like you to automatically apply any of the suggested refactors.
