# Rule: No Console Logs in Production Code

`console.log`, `console.warn`, `console.error`, and all other `console.*` methods are **forbidden** in production source code. They are only acceptable in standalone scripts, quick local experiments, or test debug sessions that are never committed.

## Why
- `console.*` has no log levels, no structured output, and no correlation IDs.
- It leaks implementation details and potentially sensitive data to stdout in production.
- It cannot be silenced, sampled, or routed without patching the runtime.
- It makes log aggregation tools (Datadog, CloudWatch, Loki, etc.) useless.

## Rules

### ❌ Never do this in committed code:
```ts
console.log('User created:', user);
console.error('Something went wrong', error);
console.warn('Deprecated call');
```

### ✅ Always use the project's logger instead:
```ts
logger.info('User created', { userId: user.id });
logger.error('Failed to create user', { error: error.message, stack: error.stack });
logger.warn('Deprecated endpoint called', { path: req.path });
```

## What to Use Instead
- If the project has a logger module (e.g., `src/shared/logger`, `@/infrastructure/logger`), **always import and use it**.
- If no logger exists yet, recommend creating one using a structured logging library (e.g., `pino`, `winston`, `bunyan` for Node.js; `loguru` for Python).
- The logger must support **log levels** (`debug`, `info`, `warn`, `error`) and **structured JSON output**.

## Agent Enforcement
- When writing new code, never emit `console.*` calls.
- When reviewing existing code, flag every `console.*` as a 🔴 Blocker.
- When asked to debug, use the logger at `debug` level — never `console.log`.
