---
description: Analyzes project dependencies for duplicates, deprecations, and anti-patterns
---
# Check Dependencies Workflow

Follow these steps to thoroughly analyze the project's dependencies:

1. **Locate Dependency Files**: 
   - Identify the package manager being used (e.g., `package.json` for npm/yarn/pnpm, `requirements.txt` or `pyproject.toml` for Python, `pom.xml` for Java, `go.mod` for Go).
   - Use the `view_file` tool to extract the list of installed dependencies.

2. **Analyze for Duplication**:
   - Review the list and identify if there are multiple libraries that serve the exact same core purpose.
   - *Example:* Using both `moment` and `date-fns` for dates, or `axios` and `node-fetch` for HTTP requests, or `lodash` and `underscore`.
   - If duplicates are found, recommend the most modern, actively maintained, or preferred library and suggest removing the others.

3. **Check for Deprecations and Maintenance Status**:
   - Check if any libraries are officially deprecated (e.g., `request`, `tslint`, `moment`).
   - Identify packages that are widely considered unmaintained, obsolete, or those that have not received context updates in years.
   - *Action:* Provide a recommendation for the modern standard replacement for any deprecated library.

4. **Identify Anti-Patterns / Not Recommended Uses**:
   - Spot heavy or bloated libraries that could easily be replaced by native language/framework features (e.g., using `lodash` when modern native array methods suffice, or using a large UI component library for a single button).
   - Flag libraries that pose potential security risks, have known significant performance bottlenecks, or violate best practices for the current architecture.

5. **Review Dependency Types**:
   - Ensure development dependencies (e.g., testing frameworks, linters, build tools) are strictly separated from production dependencies correctly in the respective file format.

6. **Generate Report**: 
   - Present a detailed, structured markdown report summarizing:
     - 🚨 **Deprecated Packages:** The package name, why it's deprecated, and migration suggestions.
     - ⚠️ **Duplicate Functionalities:** The conflicting libraries and a consolidation recommendation.
     - 💡 **Modernization Tips:** Suggestions to replace heavy libraries with lighter alternatives or native features.
     - 📦 **Structure & Cleanliness:** Any misclassified dev/prod dependencies.
   - *Wait for User Action:* Ask the user if they'd like me to assist in cleaning up, replacing, or migrating any of the flagged dependencies.
