---
description: Intelligently populates test data into the database by analyzing dependencies and prompting the user for missing information.
---

# Smart Data Seeder (/smart-seed)

This workflow analyzes a specific entity or resource in the application and generates coherent test data, respecting its relationships, validations, and constraints.

## Instructions for the Agent:

When the user invokes this workflow with a target resource (e.g., `/smart-seed Quiz`, `/smart-seed entity="User"`), you must strictly follow these steps:

1.  **Analyze the Resource and its Dependencies:**
    *   Search the codebase (models, database schemas, TypeScript interfaces, etc.) to understand how the requested entity or resource is defined.
    *   Identify all required and optional properties.
    *   **Identify Relationships (Foreign Keys):** If the resource requires other entities to exist (e.g., a `QuizAttempt` requires a `Quiz` and a `User`), note these dependencies.

2.  **Verify Existing Data (If Applicable):**
    *   Check (if possible based on current implementation, e.g., by looking at previous seed scripts or querying the DB if tools are available) whether the required dependencies already exist.

3.  **Interact with the User (Data Gathering):**
    *   Present the user with a brief summary of the resource structure and any dependencies you found.
    *   Ask the user:
        *   "How many records would you like to generate for this resource?"
        *   If there are missing dependencies: "I detected that to create [Resource], I need [Dependency]. Do you want me to automatically generate mock data for [Dependency] as well, or will you provide it?"
        *   "For text fields (e.g., questions, descriptions), do you have a specific theme in mind, or should I generate them randomly based on the app's core domain (AI software development)?"
    *   **WAIT for the user's response before proceeding to step 4.**

4.  **Generate and Populate (Script Creation):**
    *   Using the user's answers, generate a database seeding script (e.g., in TypeScript or SQL, depending on the project's stack).
    *   Ensure the script handles the insertion of dependencies in the correct order (e.g., insert `Quiz` first, then `Questions`, then `Options`).
    *   Use the application's data access layer models or repositories.
    *   Save this file appropriately (e.g., in `prisma/seeds/`, `database/seeds/`, or `/tmp/`).

5.  **Execute and Report:**
    *   Provide the user with the exact command to run the generated script (e.g., `npx ts-node path/to/seed-script.ts`).
    *   If the user approves, execute the script or assist them in running it.
    *   Confirm success by printing a sample of the newly inserted data.
