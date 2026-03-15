# Rule: Module Import Boundaries

This rule enforces strict boundaries and clear dependency paths across the codebase by differentiating how internal module code connects versus how external modules communicate.

### 1. Internal Imports (Same Module)
**RULE: ALWAYS use relative imports (`./`, `../`)**
When importing a file that belongs to the *same* feature or module context, use relative paths.
- **Why?** It keeps the module cohesive and portable. If the entire module is moved to another directory, its internal references remain intact without breaking.
- **Example:** `import { calculateTotal } from './calcs'` or `import { UserType } from '../types'`

### 2. External Imports (Different Module)
**RULE: ALWAYS use path aliases (e.g., `@/`, `~`)**
When a module needs to pull in logic, types, or utilities from an entirely *different* module or from shared directories, you must use an alias. **NEVER** use long relative paths (e.g., `../../../../user/types`) for cross-module or global dependencies.
- **Why?** It prevents "path hell", clarifies at a glance that the dependency is external/global, and safely decouples location logic across large domains.
- **Example:** `import { UserService } from '@/modules/user/services'` or `import { formatDate } from '@/shared/utils'`

### 3. Barrel Files (`index.ts`) — Module Public API Only
**RULE: ONE barrel file per module maximum, used exclusively to define its public contract.**
Barrel files (`index.ts` / `index.js`) are only allowed at the **root** of a module folder to expose its public API. They must **never** be used as folder organizers inside subfolders.

**✅ DO:**
- Create ONE `index.ts` at the module root that exports only what other modules are *allowed* to consume.
- When importing from another module via alias, prefer importing from its barrel: `import { UserService } from '@/modules/user'`
- Keep the barrel lean — only explicitly list the public exports, do not re-export everything blindly.

**❌ DON'T:**
- Create `index.ts` files inside every subfolder just to avoid typing the full filename.
- Re-export internal implementation details from the barrel (keeps internals private).
- Use the barrel to import *within your own module* — always use direct relative imports internally.

**Why?** Barrel files used correctly enforce module encapsulation (consumers don't know your internal structure) and enable safe refactoring. Used incorrectly (in every subfolder), they cause circular dependency bugs, break tree-shaking, and obscure where code actually lives — making navigation harder for both developers and AI tools.
