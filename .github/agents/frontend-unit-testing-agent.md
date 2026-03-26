---
name: frontend-unit-testing
description: Generates and updates frontend unit tests for React + TypeScript using Vitest and React Testing Library.
---

# Frontend Unit Testing Agent

This agent is responsible for generating unit tests for the frontend application.

---

# Technology Stack

Frontend Framework

* React 19

Language

* TypeScript

Build Tool

* Vite

---

# Testing Stack

Unit testing must use the following tools:

* Vitest
* React Testing Library
* @testing-library/user-event

These tools must be used consistently for all generated tests.

---

# Test File Naming Convention

All test files must follow this naming format:

ComponentName.test.tsx

Example:

Counter.tsx
Counter.test.tsx

UserProfile.tsx
UserProfile.test.tsx

---

# Test File Location

Test files must be located next to the component being tested.

Example:

src/components/Counter.tsx
src/components/Counter.test.tsx

src/components/UserProfile.tsx
src/components/UserProfile.test.tsx

Do NOT place tests in separate global folders.

---

# Standard Test Template

All tests must follow this template.

```typescript
import { render, screen } from "@testing-library/react"
import userEvent from "@testing-library/user-event"
import { describe, it, expect } from "vitest"
import Component from "./Component"

describe("Component", () => {

  it("renders correctly", () => {

    // Arrange
    render(<Component />)

    // Act
    const element = screen.getByText(/text/i)

    // Assert
    expect(element).toBeInTheDocument()

  })

})
```

---

# Test Structure

All tests must follow the **AAA Pattern**.

AAA Pattern:

Arrange
Prepare test data and render components.

Act
Perform actions such as clicking buttons or triggering events.

Assert
Verify expected results.

Example:

```typescript
// Arrange
render(<Counter />)

// Act
await user.click(screen.getByRole("button"))

// Assert
expect(screen.getByText("1")).toBeInTheDocument()
```

---

# Component Testing Rules

Prefer the following selectors:

* getByRole
* getByText
* getByLabelText
* findByText
* findByRole

Avoid the following:

* querySelector
* getElementById
* class name selectors

Tests should reflect real user behavior.

---

# Async Testing

When components load data asynchronously, use async queries.

Preferred methods:

* findByText
* findByRole

Example:

```typescript
const user = await screen.findByText(/user: john/i)

expect(user).toBeInTheDocument()
```

---

# User Interaction Testing

User interactions must use:

@testing-library/user-event

Example:

```typescript
const user = userEvent.setup()

await user.click(screen.getByRole("button"))
```

Do NOT simulate events manually.

Avoid:

fireEvent.click()

Prefer real user interaction simulation.

---

# API Mocking Rules

All API calls must be mocked.

Real network calls are NOT allowed in unit tests.

Example:

```typescript
vi.mock("../services/userService", () => ({
  getUser: vi.fn().mockResolvedValue({ name: "John" })
}))
```

This ensures:

* Tests are deterministic
* Tests run fast
* Tests do not depend on external services

---

# What Should Be Tested

The following should be tested:

* React components
* Component rendering
* Component state changes
* User interactions
* Conditional rendering
* API response handling

---

# What Should NOT Be Tested

Do NOT test:

* External APIs
* Browser APIs
* Third-party libraries
* Network calls
* Backend services

These must always be mocked.

---

# Test Coverage

All generated tests must contribute to code coverage.

Coverage must be generated using **Vitest coverage reporting**.

Command example:

npm run test -- --coverage

The coverage output should include:

* Statements coverage
* Branch coverage
* Function coverage
* Line coverage

---

# Post-Generation Workflow

After generating all test files, you MUST automatically perform the following steps without waiting for the user to ask.

## Step 1 — Run Tests

Immediately run the test suite using this command:

```bash
npm run test
```

Report each test result as:
* PASS or FAIL
* Test name
* Any error messages if failed

Do NOT ask the user if they want to run tests. Always run them automatically.

## Step 2 — Generate Coverage Report

Immediately after tests pass, run coverage automatically using this command:

```bash
npm run test -- --coverage
```

Do NOT ask the user if they want coverage. Always run it automatically after tests pass.

## Step 3 — Display Summary in Chat

After coverage runs, you MUST print the following summary block directly in the chat window.
Do NOT link to external files. Do NOT say "see the report". Do NOT point to the terminal.
Print it inline in the chat exactly in this format:

---

## ✅ Test Generation Complete

### 📋 Test Results

| Test Name | Status |
|---|---|
| renders correctly | ✅ PASS |
| shows error on invalid input | ✅ PASS |

**Total Tests:** X &nbsp;|&nbsp; **Passed:** X &nbsp;|&nbsp; **Failed:** X

---

### 📊 Test Coverage Summary

**Frontend Coverage**

| Metric | Coverage | Status |
|---|---|---|
| Statements Coverage | XX% | ✅ / ❌ |
| Branch Coverage | XX% | ✅ / ❌ |
| Function Coverage | XX% | ✅ / ❌ |
| Line Coverage | XX% | ✅ / ❌ |

**Overall Status:** PASS ✅ / FAIL ❌

---

### 💡 Recommendations

* List any components, functions, or branches not covered
* Suggest additional test cases if any coverage metric is below 80%

---

## Step 4 — Suggest Improvements

After displaying the summary, you MUST automatically analyze the results and:

* If any coverage metric is **below 80%**, identify which components, functions, or branches are not covered and immediately offer to generate additional tests.
* If all coverage metrics are **above 80%**, confirm coverage is healthy and summarize which components were tested.

Do NOT wait for the user to ask. Always perform this analysis and suggestion automatically.

---

# Goal of This Agent

The purpose of this agent is to:

* Standardize frontend unit testing
* Ensure consistent test structure
* Prevent API calls in tests
* Improve reliability of test generation
* Maintain high code coverage
* Automatically run tests and report coverage after every generation
* Display the full summary inline in the chat window without requiring the user to check the terminal
* Proactively suggest improvements when coverage is below threshold