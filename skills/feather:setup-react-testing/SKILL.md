---
name: feather:setup-react-testing
description: Adds React testing environment to an existing TDD Guard setup. Installs @testing-library/react, jsdom, @vitejs/plugin-react, and configures Vitest for React component testing. REQUIRES feather:setup-tdd-guard to be completed first. Use when setting up React testing, adding testing-library, configuring jsdom, or enabling component testing.
---

# Set Up React Testing Environment

## Overview

Extends the core TDD Guard setup with React-specific testing dependencies and configuration. Enables component testing with @testing-library/react in a jsdom environment.

**This is an add-on skill. Run `/feather:setup-tdd-guard` first.**

## When to Use

- React projects that need component testing
- After completing `/feather:setup-tdd-guard` core setup
- When you need @testing-library/react, jsdom, or React plugin in vitest

## Resume Detection

If `vitest.config.ts` already contains `react()` plugin AND `environment: "jsdom"`, steps 1-3 are already done. Skip directly to **Step 5 (Smoke Test)**.

## Git Workflow

**Before starting:**
```bash
git status  # Must be clean (no uncommitted changes)
```

**After smoke test passes (step 5):**
```bash
git add vitest.config.ts src/test/setup.ts package.json package-lock.json
git commit -m "Add React testing environment"
```

## Prerequisites

**CRITICAL:** This skill requires `/feather:setup-tdd-guard` to be fully completed (including smoke test). Verify:

```bash
grep -q "tdd-guard" .claude/settings.json && echo "TDD Guard hooks: OK" || echo "MISSING — run /feather:setup-tdd-guard first"
grep -q "VitestReporter" vitest.config.ts && echo "VitestReporter: OK" || echo "MISSING — run /feather:setup-tdd-guard first"
```

If any check fails, **STOP** and complete `/feather:setup-tdd-guard` first.

## Installation Steps

### 1. Install React Testing Dependencies

```bash
npm install -D jsdom @testing-library/react @testing-library/jest-dom @vitejs/plugin-react
```

### 2. Update vitest.config.ts

Add these to the existing core config (do not overwrite — merge into what's there):

**Add import at the top:**
```typescript
import react from "@vitejs/plugin-react";
```

**Add to `defineConfig` (top level, before `test`):**
```typescript
plugins: [react()],
```

**Add inside `test`:**
```typescript
environment: "jsdom",
setupFiles: ["./src/test/setup.ts"],
```

**Add `"src/test/"` to `coverage.exclude`** (to ignore setup files from coverage).

**Complete config for reference** — if starting from the core TDD Guard config, the final result should look like:

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import { VitestReporter } from "tdd-guard-vitest";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    reporters: ["default", new VitestReporter(__dirname)],
    setupFiles: ["./src/test/setup.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: [
        "node_modules/",
        "src/test/",
        "**/*.test.{ts,tsx,js,jsx}",
        "**/*.config.{ts,js}",
      ],
      thresholds: {
        lines: 100,
        branches: 100,
        functions: 100,
        statements: 100,
      },
    },
  },
});
```

### 3. Create Test Setup File

Create `src/test/setup.ts`:

```typescript
import "@testing-library/jest-dom/vitest";
```

This registers jest-dom matchers (`toBeInTheDocument`, `toHaveTextContent`, etc.) globally for all tests.

### 4. Pre-flight Checks

Run this to verify all React testing pieces are in place:

```bash
echo "=== React Testing Pre-flight ===" && PASS=0 && FAIL=0 && \
check() { if eval "$2" > /dev/null 2>&1; then echo "  PASS: $1"; PASS=$((PASS+1)); else echo "  FAIL: $1"; FAIL=$((FAIL+1)); fi; } && \
check "@testing-library/react installed" "[ -d node_modules/@testing-library/react ]" && \
check "@testing-library/jest-dom installed" "[ -d node_modules/@testing-library/jest-dom ]" && \
check "@vitejs/plugin-react installed" "[ -d node_modules/@vitejs/plugin-react ]" && \
check "jsdom installed" "[ -d node_modules/jsdom ]" && \
check "vitest.config.ts has react plugin" "grep -q 'react()' vitest.config.ts" && \
check "vitest.config.ts has jsdom" "grep -q 'jsdom' vitest.config.ts" && \
check "Test setup file exists" "[ -f src/test/setup.ts ]" && \
check "Setup file has jest-dom import" "grep -q 'jest-dom' src/test/setup.ts" && \
echo "" && echo "Results: $PASS passed, $FAIL failed" && \
if [ $FAIL -eq 0 ]; then echo "Pre-flight complete."; else echo "Fix failures before continuing."; fi
```

If any check fails, go back and fix the corresponding step. Do not continue to step 5.

### 5. Smoke Test

Verify React component rendering works end-to-end (jsdom + React plugin + testing-library + jest-dom matchers).

**Step 1.** Create a test component using Bash:

```bash
mkdir -p src/__tests__
cat > src/__tests__/react-smoke.test.tsx << 'EOF'
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';

function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}

describe('React Testing Smoke', () => {
  it('renders a component with jest-dom matcher', () => {
    render(<Greeting name="World" />);
    expect(screen.getByText('Hello, World!')).toBeInTheDocument();
  });
});
EOF
```

**Step 2.** Run vitest:

```bash
npx vitest run src/__tests__/react-smoke.test.tsx 2>&1
```

- If passes → **PASS.** Print: `"Smoke test PASSED — React component rendering works."`
- If fails → **FAIL. Do not commit.**
  Check: Is `@vitejs/plugin-react` in plugins? Is `environment: "jsdom"` set? Does `src/test/setup.ts` have the jest-dom import?

**Step 3.** Clean up:

```bash
rm -f src/__tests__/react-smoke.test.tsx
rmdir src/__tests__ 2>/dev/null || true
```

**Setup is complete only when the smoke test passes. Then commit (see Git Workflow above).**

## Convex Projects

If using Convex, add `convex/_generated/` to your coverage exclusions in `vitest.config.ts`:

```typescript
coverage: {
  exclude: [
    "node_modules/",
    "src/test/",
    "**/*.test.{ts,tsx,js,jsx}",
    "**/*.config.{ts,js}",
    "convex/_generated/",  // Add this line
  ],
}
```

## File Structure After Setup

On top of the core TDD Guard files, this skill adds:

```
project/
├── vitest.config.ts           # Updated with React plugin + jsdom
└── src/
    └── test/
        └── setup.ts           # jest-dom matchers
```

## Troubleshooting

### Wrong environment errors
- Ensure `environment: "jsdom"` is set in vitest.config.ts
- If you have mixed test types (unit + integration), use `environmentMatchGlobs` to apply jsdom only to component tests

### jest-dom matchers not available
- Verify `src/test/setup.ts` exists with `import "@testing-library/jest-dom/vitest"`
- Verify `setupFiles: ["./src/test/setup.ts"]` is in vitest.config.ts

### React plugin errors
- Ensure `@vitejs/plugin-react` is installed: `npm ls @vitejs/plugin-react`
- Ensure `plugins: [react()]` is at the top level of defineConfig, not inside `test`
