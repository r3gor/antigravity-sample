---
description: Given a file or module, generates comprehensive unit and integration tests following the project's testing framework, coverage best practices, and architectural boundaries.
---

# Create Tests (/create-tests)

This workflow acts as a senior QA engineer. Given a target file, function, class, or module, it analyzes the implementation and produces comprehensive, maintainable, and meaningful tests.

## Instructions for the Agent:

When the user invokes this workflow (e.g. `/create-tests src/modules/user/UserService.ts`), strictly follow these steps:

### Step 1: Identify the Target
- Read the file or files the user specified.
- If no file is specified, ask: *"Which file or module would you like me to write tests for?"*
- Identify the public interface: exported functions, classes, and methods that need testing.
- Note any dependencies (injected services, repositories, external APIs) that will need to be mocked.

### Step 2: Detect the Testing Stack
- Scan `package.json`, `pyproject.toml`, or equivalent for the test framework in use:
  - JavaScript/TypeScript: Jest, Vitest, Mocha
  - Python: Pytest
  - Java: JUnit
  - Go: `testing` package
- Identify the assertion library and mocking library (e.g., `jest.fn()`, `sinon`, `unittest.mock`).
- Find existing test files in the project to understand conventions (file placement, naming, structure).

### Step 3: Design the Test Plan
Before writing any code, present a test plan to the user:

```
📋 Test Plan for: UserService

Unit Tests:
  ✅ createUser — success path
  ✅ createUser — throws UserAlreadyExistsError when email is taken
  ✅ createUser — throws ValidationError when email is invalid
  ✅ getUserById — returns user when found
  ✅ getUserById — throws UserNotFoundError when id does not exist
  ✅ deleteUser — calls repository.delete with correct id

Integration Tests (if applicable):
  ✅ createUser — persists user in the database
  ✅ getUserById — retrieves the correct user from the database
```

> **Pause here.** Ask the user: *"Does this test plan look complete? Are there any edge cases or scenarios I should add before I start writing?"*

### Step 4: Write the Tests
Once the plan is approved, generate the test file(s) following these rules:

**Structure:**
- Group tests by method/function using `describe` blocks.
- Use `it` or `test` with a clear description in the format: `'should <expected behavior> when <condition>'`
- Follow the **AAA pattern**: Arrange → Act → Assert. Separate each section with a blank line.

**Mocking:**
- Mock **all external dependencies** (repositories, external APIs, loggers, config). Unit tests must test logic in isolation.
- Do NOT mock the system under test itself.
- For hexagonal architecture projects: mock at the Port level, never at the infrastructure level.

**Coverage priorities:**
1. 🟢 **Happy path** — the main success scenario.
2. 🔴 **Error paths** — every exception or rejection the code can throw.
3. 🟡 **Edge cases** — empty arrays, null values, boundary values, empty strings.
4. 🔵 **Side effects** — verify that collaborators (mocks) were called with the correct arguments.

**Example test structure (TypeScript/Jest):**
```ts
describe('UserService', () => {
  let userService: UserService;
  let userRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    userRepository = {
      findByEmail: jest.fn(),
      save: jest.fn(),
    } as jest.Mocked<UserRepository>;
    userService = new UserService(userRepository);
  });

  describe('createUser', () => {
    it('should create and return a user when email is not taken', async () => {
      // Arrange
      userRepository.findByEmail.mockResolvedValue(null);
      userRepository.save.mockResolvedValue({ id: '1', email: 'test@example.com' });

      // Act
      const result = await userService.createUser({ email: 'test@example.com' });

      // Assert
      expect(result.email).toBe('test@example.com');
      expect(userRepository.save).toHaveBeenCalledTimes(1);
    });

    it('should throw UserAlreadyExistsError when email is already registered', async () => {
      // Arrange
      userRepository.findByEmail.mockResolvedValue({ id: '1', email: 'test@example.com' });

      // Act & Assert
      await expect(
        userService.createUser({ email: 'test@example.com' })
      ).rejects.toThrow(UserAlreadyExistsError);
    });
  });
});
```

### Step 5: Place the Test File
- Follow the project's existing convention for test file location.
- If no convention is found, place the test file alongside the source file with a `.spec.ts` or `.test.ts` suffix.
- Always show the user the final path where the file was created.

### Step 6: Run the Tests
- Run the test suite for the newly created file: `npx jest <filename>` or equivalent.
- If tests fail, analyze the error, fix the test or implementation (flagging which), and re-run.
- Report the final test results: number of tests passed, coverage (if available).
