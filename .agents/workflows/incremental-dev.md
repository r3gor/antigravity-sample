---
description: implements a feature incrementally. It starts with a basic, functional Proof of Concept (POC), asks for user feedback, and then refines it to production-ready code.
---

# Incremental Feature Development (/incremental-dev)

This workflow is designed to build features progressively. It favors rapid prototyping to validate the core idea (Proof of Concept) before investing time in complex architectural layers, error handling, and tests.

## Instructions for the Agent:

When the user invokes this workflow (e.g., `/incremental-dev "add a quiz scoring mechanism"`), you must strictly follow these phases in order. **Never skip a phase or assume approval.**

### Phase 1: Proof of Concept (POC) Construction
1.  **Understand the Core Goal:** Analyze the user's request to identify the bare minimum functionality required to prove the concept works.
2.  **Build the Happy Path:** Implement the feature targeting *only* the successful execution path. 
    *   Prioritize speed and immediately visible results over perfect architecture.
    *   It's acceptable to bypass strict Hexagonal Architecture rules (e.g., putting logic in a controller) temporarily just for this POC phase.
    *   Skip extensive error handling, validation, and tests.
3.  **Present the POC:** Show the user the implemented code or explain how it works.
    *   Highlight what is functional and what has been intentionally left out for now.

### Phase 2: User Validation (Wait State)
1.  Ask the user for explicit feedback: 
    *   *"Does this basic functionality align with what you want?"*
    *   *"Are there any core behavior changes needed before we make it production-ready?"*
2.  **STOP AND WAIT:** Do not proceed to Phase 3 until the user explicitly approves the POC or requests changes. If changes are requested, iterate on Phase 1.

### Phase 3: Production Refinement
*Only proceed here after User Validation.*
1.  **Apply Architecture:** Refactor the POC code to strictly follow the project's architecture (e.g., Hexagonal Architecture). Separate the domain logic from the framework/infrastructure.
2.  **Robust Error Handling:** Add comprehensive input validation, error catching, and meaningful HTTP status codes or domain error classes.
3.  **Security & Edge Cases:** Handle null values, unauthorized access attempts, and other potential edge cases.
4.  **Testing Strategy:** Write unit tests for the core business logic or outline the tests that must be written.

### Phase 4: Final Polish
1.  Review against standard design principles (SOLID).
2.  Present the finalized, production-ready code to the user.
3.  Provide a brief summary of the transformations made from the POC to the final version.
