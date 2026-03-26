# Sonar AllAgents

GitHub Copilot custom agents & prompts for enforcing quality, performance, and consistency across a **.NET / C#** backend and **React 19 / TypeScript** frontend.

---

## Agents

| Agent | File | Description |
|---|---|---|
| **Code Review** | `code-review.agent.md` | Automated PR reviews for C# .NET — 50+ checks across code quality, security, performance, .NET best practices, testing, and PR hygiene. Outputs a scored review with categorized issues. |
| **Backend Unit Testing** | `backend-unit-testing-agent.md` | Generates xUnit + Moq tests for .NET Web API. Enforces AAA pattern, `MethodName_Should_When` naming, and dependency mocking. Auto-runs tests and reports Coverlet coverage. |
| **Frontend Architect** | `frontend-architect.agent.md` | Lightweight React/TypeScript scaffold agent. Creates a simpler starter structure with core folders, API boilerplate, environment files, Helm values, service-config YAMLs, and IIS-ready `web.config`. |
| **Frontend Unit Testing** | `frontend-unit-testing-agent.md` | Generates Vitest + React Testing Library tests for React 19. Co-located test files, `userEvent` for interactions, `vi.mock()` for APIs. Auto-runs tests and reports coverage. |
| **React Project Scaffold** | `react-project-scaffold.agent.md` | Advanced Vite + React + TypeScript scaffold agent. Generates a more opinionated enterprise starter with Atomic Design, Redux Toolkit, React Router, theme/styles setup, detailed boilerplate, environment configs, Helm values, and deployment files. |
| **Quality Build** | `quality-build.agent.md` | 6-phase QA pipeline (Dependencies → ESLint → TypeScript → Tests/Coverage → Build → Report) with A+ to F grading. Auto-fixes issues up to 3 iterations. |
| **Everest Backend Architect** | `everest-backend-architect.agent.md` | Interactive .NET solution scaffolder. Uses the Everest prompt to gather architecture, API, DB, auth, testing, and DevOps choices, then generates the full backend solution and validates the build. |
| **PR Review** | `pr-review.agent.md` | Azure DevOps MCP-powered PR review agent with human-in-the-loop approval. 11-step review covering correctness, security, performance, naming, structure, tests, dependencies, and documentation. Posts reviews via MCP. |
| **Perf Detector** *(orchestrator)* | `perf-detector.agent.md` | Routes files to specialist sub-agents by type, aggregates results into a unified Performance Smell Report. Supports broad, targeted, and report-only modes. |
| **Perf Runtime Backend** | `perf-runtime-backend.agent.md` | Live backend load-testing agent. Discovers API endpoints, confirms scope with the user, builds/starts the server, runs sequential and concurrent HTTP benchmarks, compares against saved baselines, and reports latency/throughput/error metrics. |
| **Perf Runtime Frontend** | `perf-runtime-frontend.agent.md` | Live frontend runtime benchmark agent. Analyses a React component, confirms props/routes/budgets with the user, runs Vitest + React Testing Library performance checks, optionally measures bundle impact, compares baselines, and reports render/interactions/profiler metrics. |
| **Backend Unit Testing V2** | `backend-unit-test-cases-for-new-changes-v2.md` | Enhanced unit-test agent targeting. Validates & fixes existing tests first, then fills coverage gaps to reach **100% line coverage**. Uses only `Assert` class methods (no fluent assertions), prevents CS0854 errors, documents private helpers, and generates a Fine Code Coverage Report. |
| **PR Description** | `pr-description-with-ss.md` | Automates PR description creation. Collects a one-line summary and optional JIRA ticket URL(s), compares the current branch against a selected/parent branch, analyses all git changes with commit-wise breakdown, and outputs a ready-to-copy PR description with screenshot placeholders. |

### Backend Unit Testing — V1 vs V2

| | V1 (`backend-unit-testing-agent`) | V2 (`backend-unit-test-cases-for-new-changes-v2`) |
|---|---|---|
| **Scope** | Generates tests for any given service/class | Targets only new/changed code |
| **Existing tests** | Does not inspect existing tests | Runs existing tests first; fixes failures before adding new ones |
| **Coverage target** | Best-effort | Mandatory **100% line coverage** |
| **Assertion style** | Allows fluent assertions (`Should`) | `Assert.*` only (no fluent assertions) |
| **Error prevention** | Basic | CS0854 optional-parameter error prevention built-in |
| **Reporting** | Coverage via Coverlet | Fine Code Coverage Report + detailed summary tables |
| **Private methods** | Not documented | Mandatory XML doc comments on private helpers |

- `@backend-unit-testing` → quick test generation for any class
- `@backend-unit-test-cases-for-new-changes-v2` → thorough coverage enforcement for PR-ready changes

### Frontend Scaffolding Agents — Basic vs Advanced

| Agent | Level | When to Use |
|---|---|---|
| **Frontend Architect** | Basic | Quick project bootstrap with core folders, API setup, env files, Helm values, and deployment basics. No Atomic Design, Redux, or Router setup. |
| **React Project Scaffold** | Advanced | Full enterprise starter with Atomic Design, Redux Toolkit, React Router, theme/styles, TypeScript/Vite configs, folder READMEs, and broad boilerplate. |

- `@frontend-architect` → lightweight, fewer generated files, faster setup
- `@react-project-scaffold` → comprehensive, opinionated, production-ready structure

### Performance Sub-Agents

These are invoked automatically by the **Perf Detector** orchestrator — not called directly.

For live runtime benchmarking, invoke the standalone runtime agents directly:
- `@perf-runtime-backend`
- `@perf-runtime-frontend`

| Sub-Agent | File | Detects |
|---|---|---|
| **N+1 Query** | `perf-n-plus-one-detector.agent.md` | 7 patterns — DB call in loop, missing Include, Map vs ProjectTo, MediatR Send in loop, FluentValidation async DB, correlated subquery, cartesian explosion |
| **Unbounded Query** | `perf-unbounded-query-detector.agent.md` | 5 patterns — ToListAsync without pagination, full table load, unbounded Include, API without paging, count via materialization |
| **Missing Index** | `perf-missing-index-detector.agent.md` | 6 patterns — FK without index, status/enum filter, OrderBy on non-PK, composite filter, unique lookup, missing HasIndex in EF config |
| **Large Object** | `perf-large-object-detector.agent.md` | 6 patterns — large materialization, string concat in loop, Map on large collection, file to byte[], full entity fetch, List\<T\> for streaming |
| **Re-render Loop** | `perf-rerender-detector.agent.md` | 8 patterns — useEffect object dep, missing deps, object-in-body props, useSelector new ref, missing memo, inline fn to memo child, setState cascade, dispatch during render |

All sub-agents provide severity ratings, copy-paste-ready fixes, and before/after comparisons (SQL or render counts).

---

## How Runtime Performance Testing Works

### Backend runtime testing
The backend runtime agent does **live API performance testing**, not unit testing.

It uses:
- **PowerShell**
- **.NET `HttpClient`**
- **real HTTP requests** against the running API
- a generated temporary **PowerShell script** executed from the terminal

#### Backend runtime flow
1. Reads backend source to discover endpoints such as `MapGet`, `MapPost`, `[HttpGet]`, and `[HttpPost]`
2. Finds the server URL/port from project configuration
3. Presents detected endpoints to the user for confirmation
4. Asks for:
   - endpoints to test
   - concurrency level
   - request count
   - query/body values
   - baselines
   - performance budgets
   - permission to build and start the server
5. Builds and starts the API
6. Sends warm-up, sequential, and concurrent requests
7. Measures:
   - latency (`min`, `avg`, `p50`, `p95`, `p99`, `max`)
   - throughput
   - error rate
   - response payload size
8. Stops the server and writes benchmark summaries under `.benchmarks/backend/`

### Frontend runtime testing
The frontend runtime agent does **component runtime benchmarking** using:
- **Vitest**
- **React Testing Library**
- **jsdom**
- the existing `npm run perf:component` script

It measures:
- mount time
- rerender time
- unmount time
- React Profiler timings
- interaction latency
- DOM size
- console errors/warnings
- optional bundle impact

### Backend vs Frontend runtime approach

| Area | Runtime method |
|---|---|
| **Backend** | Live HTTP load testing with **PowerShell + .NET `HttpClient`** |
| **Frontend** | Component benchmarking with **Vitest + React Testing Library + jsdom** |

### Important distinction
- **Backend Unit Testing Agent** = generates **xUnit + Moq** tests
- **Perf Runtime Backend Agent** = runs **live HTTP performance benchmarks**
- **Perf Runtime Frontend Agent** = runs **live component performance benchmarks**

---

## Prompts

| Prompt | File | Description |
|---|---|---|
| **Create Everest Solution** | `create-everest-solution.prompt.md` | 2100+ line prompt used by the Everest Backend Architect agent. Defines 7 rounds of questions, variable resolution, full file generation (Core → Application → Infrastructure → API → Tests → CI/CD → Docker → Helm), and build validation. Supports REST/GraphQL/gRPC, Monolith/Microservices, multiple DB/auth/test combos. |

---

## Tech Stack

| | |
|---|---|
| **Backend** | .NET 8/10, C#, ASP.NET Core, EF Core, PostgreSQL, SQL Server, MediatR/CQRS, FluentValidation, AutoMapper |
| **Frontend** | React 19, TypeScript, Vite, Redux Toolkit, MUI v5 |
---

## Getting Started

1. Clone this repo and open in **VS Code** with **GitHub Copilot** enabled.
2. Agents under `.github/agents/` are automatically available in Copilot Chat.
3. Invoke by name:
   ```
   @backend-unit-testing generate tests for UserService
   @code-review review the current PR
   @frontend-architect scaffold a frontend starter
   @frontend-unit-testing generate tests for LoginForm
   @quality-build check if build is PR ready
   @react-project-scaffold create a new React app scaffold
   @everest-backend-architect create a new solution
   @perf-detector scan this file for performance issues
   @perf-runtime-backend load test the API
   @perf-runtime-frontend benchmark this React component
   @backend-unit-test-cases-for-new-changes-v2 comprehensive xUnit unit test cases with MANDATORY 100% code coverage
   @pr-description-with-ss prepare PR description
   ```

> The five `perf-*-detector` sub-agents are dispatched automatically by the orchestrator and are not invoked directly.
> To invoke them remove `user-invokable: false`
