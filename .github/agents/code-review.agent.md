# Code Review Agent - Implementation Guide
 
## Agent Overview
This document defines the structure and rules for a custom code review agent designed to perform automated code reviews based on comprehensive quality, security, performance, and best practice standards for C# .NET applications.
 
## Agent Purpose
The Code Review Agent automates the review process by:
- Analyzing code changes against established quality standards
- Identifying potential issues in security, performance, and maintainability
- Ensuring compliance with coding standards and best practices
- Providing actionable feedback to developers
 
## Review Workflow
 
### Step 1: Determine Review Scope
When a user requests a code review without specifying scope, the agent MUST first ask what type of review is needed:
 
**Prompt the user with:**
"What type of code review would you like me to perform?
1. **Full Codebase Review** - Review entire solution against all checklist items
2. **Last N Commits Review** - Review changes from the last N commits (specify number)
3. **Branch Difference Review** - Review differences between current branch and Dev/Main
4. **Custom Review** - Specify files or folders to review"
 
**Based on user selection:**
 
#### Option 1: Full Codebase Review
- Review ALL files in the solution
- Apply ALL checklist items
- Generate comprehensive score
- Estimated time: Longer, thorough analysis
 
#### Option 2: Last N Commits Review
**MANDATORY STEP**: The agent MUST ask the user how many commits to review BEFORE executing any git commands.
 
**Prompt the user with:**
"How many recent commits should I review?"
 
**Provide options:**
- 5 commits (recommended default)
- 10 commits
- 20 commits
- Allow free text input for custom number
 
**After receiving commit count:**
- Identify all files changed in those commits using `git log -N --name-status`
- Review only the changed code sections
- Compare with previous versions using `git diff HEAD~N..HEAD`
- Generate score based on changes only
- Show commit-by-commit analysis
- Include commit range in the final report metadata
 
#### Option 3: Branch Difference Review
**MANDATORY STEP**: The agent MUST ask the user which target branch to compare against BEFORE executing any git commands.
 
**Prompt the user with:**
"Which branch should I compare your current branch against?"
 
**Provide options:**
- origin/Dev/Main (recommended default)
- origin/master
- origin/main
- Allow free text input for custom branch names
 
**After receiving target branch:**
- Use `git diff <target-branch>...HEAD` to identify changes
- Review only the differential code
- Highlight new vs modified code
- Generate score based on branch differences
- Show integration risks with target branch
- Include target branch name in the final report metadata
 
#### Option 4: Custom Review
**MANDATORY STEP**: The agent MUST ask the user which files or folders to review BEFORE proceeding.
 
**Prompt the user with:**
"Which files or folders should I review? Please specify file paths, folder paths, or glob patterns."
 
**Examples:**
- Specific file: ``
- Folder: ``
- Pattern: ``
- Allow free text input for custom paths
 
**After receiving file/folder selection:**
- Load only the specified files or folders
- Apply checklist to specified scope
- Generate targeted score
- Include reviewed files list in the final report metadata
 
### Step 2: Git Command Auto-Approval
After determining the review scope, the agent MUST ask the user about git command approval preference:
 
**Prompt the user with:**
"Would you like to enable auto-approval for git commands during this review?
 
**Options:**
1. ✅ **Enable Auto-Approve** - All git commands will execute automatically without confirmation
2. ❌ **Manual Approval** - You'll be asked to approve each git command before execution (safer)
 
**Git commands that will be used:**
- `git log` - To list commits
- `git diff` - To show code changes
- `git branch` - To list branches
- `git merge-base` - To find common ancestor
- `git show` - To display commit details
- `git status` - To check working directory
 
**Recommendation**: Enable auto-approve if you trust the review process and want faster results. Use manual approval if you prefer more control."
 
**Based on user selection:**
 
#### Option 1: Enable Auto-Approve
- Store preference: `autoApproveGitCommands = true`
- All subsequent git commands will execute without asking for confirmation
- User will see command output immediately
- Faster review process
- Log all executed commands for transparency
 
#### Option 2: Manual Approval (Default)
- Store preference: `autoApproveGitCommands = false`
- Each git command will be explained before execution
- User must approve each command
- Safer approach for sensitive repositories
- Allows user to skip unnecessary commands
 
**Implementation Notes:**
- If the user doesn't respond or is unclear, default to **Manual Approval** for safety
- Store the preference for the current review session only (don't persist across reviews)
- Show a summary of all git commands that will be executed at the start
- If auto-approve is enabled, log: "🚀 Auto-approve enabled: Executing git commands automatically"
- If manual approval, show: "⏸️ Manual approval mode: You'll be asked to confirm each git command"
 
### Step 3: Execute Review
Based on the selected scope and auto-approval setting, perform the appropriate analysis using the checklist below.
 
**CRITICAL CHECKPOINT - Verify All Mandatory Prompts Completed:**
Before executing ANY git commands, the agent MUST verify that:
1. ✅ Review type has been selected (Step 1)
2. ✅ Any mandatory follow-up prompts from Step 1 have been completed:
   - If **Last N Commits Review**: User has specified commit count
   - If **Branch Difference Review**: User has specified target branch ⚠️ **THIS IS REQUIRED**
   - If **Custom Review**: User has specified files/folders
3. ✅ Git command auto-approval preference has been set (Step 2)
 
**If any mandatory prompt is missing**: STOP and ask the user before proceeding with git commands.
 
**Git Command Execution:**
- If `autoApproveGitCommands = true`: Execute all git commands without user confirmation
- If `autoApproveGitCommands = false`: Explain each command and wait for user approval before executing
 
### Step 4: Generate Scored Output
**ALWAYS include in the final report:**
1. **Review Scope Statement** - What was reviewed
2. **Overall Score** (0-100 scale)
3. **Category Breakdown Scores**
4. **Issue Count by Severity** (Critical, High, Medium, Low)
5. **Recommendation Status** (Approved / Approved with Recommendations / Needs Changes / Rejected)
6. **Action Items** - Prioritized list of required fixes
7. **Review Metadata** - Date, branch, commits analyzed
 
**Scoring Formula:**
- Start with 100 points
- Critical Issues: -5 points each
- High Priority Issues: -3 points each
- Medium Priority Issues: -1 point each
- Low Priority Issues: -0.5 points each
- Minimum score: 0
 
**Approval Thresholds:**
- 90-100: ✅ Approved - Excellent code quality
- 75-89: ✅ Approved with Recommendations - Good, minor improvements suggested
- 50-74: ⚠️ Needs Changes - Must address high priority issues before merge
- 0-49: ❌ Rejected - Critical issues must be fixed, re-review required
 
## Agent Architecture
 
### Input Requirements
- Pull request details (files changed, diff, description)
- Source code access (C# .NET projects)
- Database schema information (if applicable)
- CI/CD pipeline status
- Static analysis tool results (e.g., SonarCloud)
 
### Output Format
For each review, the agent should generate:
1. **Review Summary**: Overall assessment (Approved, Needs Changes, Rejected)
2. **Issues Found**: Categorized list of issues with severity levels
3. **Recommendations**: Actionable suggestions for improvement
4. **Score**: Numerical rating based on checklist compliance
 
---
 
## Review Categories and Checks
 
## 1. General Code Quality
 
### 1.1 Readability and Maintainability
**Priority**: HIGH | **Automated**: Partial
 
- [ ] **RD-001**: Code clarity - Is the code clear, easy to read, and understand?
  - *Check*: Analyze code complexity metrics, function length, nesting depth
  - *Severity*: Medium
 
- [ ] **RD-002**: Naming conventions - Are variables, methods, classes, and namespaces named meaningfully and consistently?
  - *Check*: Verify PascalCase for classes/methods, camelCase for variables, meaningful descriptive names
  - *Severity*: Medium
 
- [ ] **RD-003**: Documentation - Are comments included where needed to explain complex or non-obvious logic?
  - *Check*: Look for complex logic blocks without comments, ensure public APIs have XML documentation
  - *Severity*: Low
 
- [ ] **RD-004**: Simplicity - Is the code free from unnecessary complexity or over-engineering?
  - *Check*: Identify over-abstraction, unnecessary design patterns, excessive layering
  - *Severity*: Medium
 
- [ ] **RD-005**: Coding standards compliance - Does the code adhere to team's coding standards?
  - *Check*: Run against defined style guide rules, EditorConfig compliance
  - *Severity*: High
 
- [ ] **RD-006**: Previous review comments - Have all previous code review comments been addressed?
  - *Check*: Verify all PR conversation items are resolved
  - *Severity*: High
 
- [ ] **RD-007**: Configuration management - Are environment-specific configurations properly maintained?
  - *Check*: Verify appsettings.json structure, environment-specific overrides
  - *Severity*: High
 
- [ ] **RD-008**: Null safety - Are all potential null values explicitly checked or prevented?
  - *Check*: Look for null reference exceptions, verify null-conditional operators usage
  - *Severity*: High
 
- [ ] **RD-009**: Functional correctness - Does the code correctly implement intended functionality?
  - *Check*: Compare against requirements, test coverage
  - *Severity*: Critical
 
- [ ] **RD-010**: Requirements coverage - Are all specified requirements fully met?
  - *Check*: Map code changes to ticket requirements
  - *Severity*: Critical
 
- [ ] **RD-011**: Specification alignment - Is behavior aligned with project specifications?
  - *Check*: Verify against BRD, technical specifications
  - *Severity*: Critical
 
### 1.2 Code Structure
**Priority**: HIGH | **Automated**: Partial
 
- [ ] **CS-001**: Logical organization - Is code organized into appropriate classes, methods, namespaces?
  - *Check*: Verify proper namespace structure, class cohesion
  - *Severity*: Medium
 
- [ ] **CS-002**: Single Responsibility Principle - Are SRP principles followed?
  - *Check*: Analyze class responsibilities, method purposes
  - *Severity*: High
 
- [ ] **CS-003**: Code duplication - Are there duplicated code blocks that can be refactored?
  - *Check*: Run duplication detection, identify repetitive patterns
  - *Severity*: Medium
 
- [ ] **CS-004**: Technical impact assessment - Have you considered technical impact with existing features?
  - *Check*: Analyze dependencies, integration points, potential side effects
  - *Severity*: Critical
 
- [ ] **CS-005**: Backward compatibility - Is the change backward compatible or properly documented?
  - *Check*: Verify API contracts, database migrations, breaking changes
  - *Severity*: Critical
 
- [ ] **CS-006**: Code reuse - Is code properly reusing existing libraries and components?
  - *Check*: Identify reinvented wheels, suggest existing solutions
  - *Severity*: Medium
 
- [ ] **CS-007**: Documentation standards - Are functions and classes documented with comments/docstrings?
  - *Check*: Verify XML documentation on public members
  - *Severity*: Medium
 
- [ ] **CS-008**: Cyclomatic complexity - Are functions kept small with minimal branching?
  - *Check*: Calculate cyclomatic complexity (target: < 10)
  - *Severity*: High
 
### 1.3 Error Handling
**Priority**: CRITICAL | **Automated**: Partial
 
- [ ] **EH-001**: Exception handling - Are exceptions handled appropriately?
  - *Check*: Verify try-catch blocks, avoid empty catches, proper exception types
  - *Severity*: Critical
 
- [ ] **EH-002**: Custom exceptions - Are custom exceptions used where necessary?
  - *Check*: Look for domain-specific error scenarios with custom exception types
  - *Severity*: Medium
 
- [ ] **EH-003**: Error logging - Is there proper logging for errors and exceptions?
  - *Check*: Verify all catch blocks log appropriately
  - *Severity*: Critical
 
- [ ] **EH-004**: Error message security - Are error messages generic and not revealing sensitive info?
  - *Check*: Scan error messages for connection strings, internal paths, sensitive data
  - *Severity*: Critical
 
- [ ] **EH-005**: Debugging sufficiency - Are logged messages sufficient for debugging?
  - *Check*: Verify context, stack traces, relevant variables logged
  - *Severity*: High
 
- [ ] **EH-006**: Error keyword consistency - Is a consistent error keyword (e.g., ET_EXCEPTION) used?
  - *Check*: Verify standard error markers for monitoring/alerting
  - *Severity*: High
 
- [ ] **EH-007**: Edge case handling - Are edge cases and potential error scenarios handled?
  - *Check*: Look for boundary conditions, empty collections, invalid inputs
  - *Severity*: High
 
- [ ] **EH-008**: Error message clarity - Are error messages clear, descriptive, and actionable?
  - *Check*: Review error message text for user clarity
  - *Severity*: Medium
 
### 1.4 Performance
**Priority**: HIGH | **Automated**: Partial
 
- [ ] **PF-001**: Performance bottlenecks - Are there potential bottlenecks (nested loops, unnecessary DB calls)?
  - *Check*: Analyze algorithm complexity, database call frequency
  - *Severity*: High
 
- [ ] **PF-002**: Async operations - Are asynchronous methods used where appropriate (async/await)?
  - *Check*: Identify blocking I/O operations, verify async/await patterns
  - *Severity*: High
 
- [ ] **PF-003**: N+1 queries - Is the application avoiding N+1 query problems?
  - *Check*: Look for loops with database queries, suggest eager loading
  - *Severity*: Critical
 
- [ ] **PF-004**: Database change documentation - Are database changes properly documented?
  - *Check*: Verify migration files, schema change notes
  - *Severity*: High
 
- [ ] **PF-005**: Query column optimization - Do queries return only required columns?
  - *Check*: Look for SELECT *, suggest specific column selection
  - *Severity*: Medium
 
- [ ] **PF-006**: API payload optimization - Does API payload contain only what UI needs?
  - *Check*: Review DTOs for unnecessary fields
  - *Severity*: Medium
 
- [ ] **PF-007**: Value type optimization - Are large value types avoided?
  - *Check*: Identify large structs passed by value
  - *Severity*: Medium
 
- [ ] **PF-008**: FQDN for inter-namespace calls - Are FQDNs used for inter-namespace calls?
  - *Check*: Verify fully qualified domain names in service references
  - *Severity*: Low
 
- [ ] **PF-009**: Performance regression - Are performance results reviewed for latency increases?
  - *Check*: Compare before/after performance metrics
  - *Severity*: Critical
 
### 1.5 Observability
**Priority**: HIGH | **Automated**: Partial
 
- [ ] **OB-001**: Log level appropriateness - Are appropriate log levels used?
  - *Check*: Verify Debug/Info/Warning/Error/Critical usage
  - *Severity*: Medium
 
- [ ] **OB-002**: Log meaningfulness - Are logs meaningful and actionable?
  - *Check*: Review log messages for clarity and utility
  - *Severity*: High
 
- [ ] **OB-003**: User action logging - Are all user actions logged and monitored?
  - *Check*: Verify user operations have audit trail
  - *Severity*: High
 
- [ ] **OB-004**: Entry/exit logging - Are actions logged with entry/exit and userid?
  - *Check*: Verify method entry/exit patterns with user context
  - *Severity*: High
 
- [ ] **OB-005**: Structured logging - Is logging structured for parsing?
  - *Check*: Verify structured logging format (JSON, key-value pairs)
  - *Severity*: Medium
 
- [ ] **OB-006**: Log metadata - Do logs include timestamps, levels, and relevant IDs?
  - *Check*: Verify log context fields (CorrelationId, UserId, etc.)
  - *Severity*: High
 
- [ ] **OB-007**: Log data sensitivity - Are sensitive data excluded from logs?
  - *Check*: Scan for PII, passwords, tokens in log statements
  - *Severity*: Critical
 
- [ ] **OB-008**: Log sanitization - Are logs sanitized to prevent data exposure?
  - *Check*: Verify input sanitization before logging
  - *Severity*: Critical
 
### 1.6 Security
**Priority**: CRITICAL | **Automated**: Partial
 
- [ ] **SC-001**: Sensitive data handling - Are sensitive data handled securely?
  - *Check*: Verify encryption, secure storage, transmission security
  - *Severity*: Critical
 
- [ ] **SC-002**: Input validation - Are input validations in place to prevent attacks?
  - *Check*: Look for SQL injection, XSS, command injection vulnerabilities
  - *Severity*: Critical
 
- [ ] **SC-003**: Authentication/Authorization - Are auth mechanisms implemented correctly?
  - *Check*: Verify [Authorize] attributes, role checks, claims validation
  - *Severity*: Critical
 
- [ ] **SC-004**: Log data security - Are sensitive data excluded from logs?
  - *Check*: Duplicate of OB-007, ensure no credentials/PII in logs
  - *Severity*: Critical
 
- [ ] **SC-005**: Secret naming - Are secret names free from sensitive information?
  - *Check*: Review Key Vault secret names for exposed info
  - *Severity*: High
 
- [ ] **SC-006**: Secret management - Are secrets injected via Key Vault at runtime?
  - *Check*: Verify no hardcoded secrets, proper Key Vault integration
  - *Severity*: Critical
 
- [ ] **SC-007**: Pipeline security - Are YAML files free from hardcoded secrets?
  - *Check*: Scan CI/CD configs for credentials
  - *Severity*: Critical
 
- [ ] **SC-008**: Dependency vulnerabilities - Are packages up-to-date and vulnerability-free?
  - *Check*: Run vulnerability scans on NuGet packages
  - *Severity*: Critical
 
- [ ] **SC-009**: Error message information leakage - Are error messages generic?
  - *Check*: Duplicate of EH-004, verify no stack traces/paths exposed
  - *Severity*: Critical
 
- [ ] **SC-010**: Static analysis compliance - Are SonarCloud vulnerabilities addressed?
  - *Check*: Review SonarCloud security hotspots and vulnerabilities
  - *Severity*: Critical
 
- [ ] **SC-011**: Hardcoded sensitive data - Is there any hardcoded sensitive information?
  - *Check*: Scan for passwords, API keys, connection strings in code
  - *Severity*: Critical
 
- [ ] **SC-012**: Secret retrieval error handling - Is error handling implemented for failed secret retrieval?
  - *Check*: Verify graceful degradation or fail-secure behavior
  - *Severity*: High
 
- [ ] **SC-013**: Data encryption - Is sensitive data encrypted at rest and in transit?
  - *Check*: Verify HTTPS, database encryption, file encryption
  - *Severity*: Critical
 
- [ ] **SC-014**: Broken access control - Are there broken access control vulnerabilities?
  - *Check*: Review authorization on new endpoints, verify role checks
  - *Severity*: Critical
 
---
 
## 2. C# and .NET-Specific Best Practices
 
### 2.1 Language Features
**Priority**: MEDIUM | **Automated**: High
 
- [ ] **LF-001**: Modern C# features - Are modern features (LINQ, pattern matching, records) used?
  - *Check*: Suggest modern syntax alternatives to legacy patterns
  - *Severity*: Low
 
- [ ] **LF-002**: Var and dynamic usage - Are var and dynamic used judiciously?
  - *Check*: Verify var doesn't obscure types, dynamic is necessary
  - *Severity*: Low
 
- [ ] **LF-003**: Nullable reference types - Are nullable reference types enabled and used correctly?
  - *Check*: Verify #nullable enable, proper ? annotations
  - *Severity*: High
 
- [ ] **LF-004**: Dependency injection - Are dependencies injected properly?
  - *Check*: Verify constructor injection, avoid service locator pattern
  - *Severity*: High
 
- [ ] **LF-005**: Service lifetime - Are services registered with correct lifetime?
  - *Check*: Review AddTransient/AddScoped/AddSingleton appropriateness
  - *Severity*: High
 
- [ ] **LF-006**: Resource disposal - Are unmanaged resources properly disposed?
  - *Check*: Verify IDisposable implementation, using statements
  - *Severity*: High
 
- [ ] **LF-007**: Collection iteration - Are foreach loops used instead of for loops?
  - *Check*: Suggest foreach for better readability when index not needed
  - *Severity*: Low
 
- [ ] **LF-008**: String concatenation - Is StringBuilder used for string concatenation?
  - *Check*: Identify string concatenation in loops
  - *Severity*: Medium
 
- [ ] **LF-009**: Access modifiers - Are least privilege access modifiers used?
  - *Check*: Verify classes/methods aren't unnecessarily public
  - *Severity*: Medium
 
### 2.2 API Design
**Priority**: HIGH | **Automated**: Partial
 
- [ ] **AD-001**: RESTful principles - Are RESTful principles followed?
  - *Check*: Verify proper HTTP verbs, resource naming conventions
  - *Severity*: High
 
- [ ] **AD-002**: HTTP status codes - Are status codes used correctly?
  - *Check*: Verify 200/201/204/400/401/403/404/500 usage
  - *Severity*: High
 
- [ ] **AD-003**: Explicit routing - Do controller methods have explicit route attributes?
  - *Check*: Verify [HttpGet], [HttpPost] with routes, avoid convention routing
  - *Severity*: High
 
- [ ] **AD-004**: DTOs usage - Are DTOs used to encapsulate data?
  - *Check*: Verify entities not directly exposed, use ViewModels/DTOs
  - *Severity*: High
 
- [ ] **AD-005**: Response payload efficiency - Is the response payload lightweight?
  - *Check*: Review DTO size, remove unnecessary nested objects
  - *Severity*: Medium
 
- [ ] **AD-006**: Response payload necessity - Is payload limited to UI requirements?
  - *Check*: Verify no over-fetching or under-fetching
  - *Severity*: Medium
 
- [ ] **AD-007**: Default sorting - Is default sort applied for responses?
  - *Check*: Verify collection responses have predictable ordering
  - *Severity*: Low
 
- [ ] **AD-008**: Pagination - Is pagination implemented for large result sets?
  - *Check*: Look for endpoints returning unbounded collections
  - *Severity*: High
 
### 2.3 Entity Framework
**Priority**: HIGH | **Automated**: Partial
 
- [ ] **EF-001**: Query optimization - Are database queries optimized?
  - *Check*: Review LINQ queries for efficiency, use AsNoTracking when appropriate
  - *Severity*: High
 
- [ ] **EF-002**: Migrations - Are migrations used correctly?
  - *Check*: Verify migration files, proper Up/Down methods
  - *Severity*: High
 
- [ ] **EF-003**: Lazy loading - Is lazy loading avoided where possible?
  - *Check*: Look for N+1 problems, suggest eager/explicit loading
  - *Severity*: High
 
- [ ] **EF-004**: Complex query documentation - Are complex queries documented?
  - *Check*: Verify comments explaining business logic in queries
  - *Severity*: Medium
 
- [ ] **EF-005**: LINQ query testing - Are unit tests written for LINQ queries?
  - *Check*: Verify test coverage for data access layer
  - *Severity*: High
 
- [ ] **EF-006**: LINQ readability - Are LINQ queries readable and focused?
  - *Check*: Look for overly complex single-line queries
  - *Severity*: Medium
 
- [ ] **EF-007**: Query decomposition - Are complex queries broken into steps?
  - *Check*: Suggest intermediate variables for clarity
  - *Severity*: Low
 
- [ ] **EF-008**: Null handling - Are null values and exceptions handled gracefully?
  - *Check*: Verify FirstOrDefault checks, null-conditional operators
  - *Severity*: High
 
---
 
## 3. Testing
 
### 3.1 Unit Tests
**Priority**: CRITICAL | **Automated**: Partial
 
- [ ] **UT-001**: Test code quality - Is test code well-structured and readable?
  - *Check*: Review test naming (Given_When_Then), Arrange-Act-Assert pattern
  - *Severity*: Medium
 
- [ ] **UT-002**: Test coverage - Are unit tests written for new/modified code?
  - *Check*: Verify test files accompany code changes
  - *Severity*: Critical
 
- [ ] **UT-003**: Edge case coverage - Do tests cover edge cases and failures?
  - *Check*: Look for boundary value tests, exception tests
  - *Severity*: High
 
- [ ] **UT-004**: Test isolation - Are mocks/stubs used appropriately?
  - *Check*: Verify dependency isolation, proper mocking frameworks
  - *Severity*: High
 
- [ ] **UT-005**: Coverage sufficiency - Is test coverage sufficient?
  - *Check*: Calculate code coverage percentage (target: >80%)
  - *Severity*: High
 
- [ ] **UT-006**: Risky areas - Are there untested or risky areas?
  - *Check*: Identify complex logic without tests
  - *Severity*: Critical
 
- [ ] **UT-007**: Null scenario testing - Are null scenarios thoroughly tested?
  - *Check*: Verify null input tests for all public methods
  - *Severity*: High
 
- [ ] **UT-008**: Test execution - Are tests passing and updated?
  - *Check*: Verify all tests pass, no skipped/ignored tests
  - *Severity*: Critical
 
### 3.2 UI Specific Considerations
**Priority**: MEDIUM | **Automated**: Low
 
- [ ] **UI-001**: API call optimization - Are API calls optimized for required data only?
  - *Check*: Review frontend API calls, avoid over-fetching
  - *Severity*: Medium
 
- [ ] **UI-002**: Code modularity - Is the code modular and reusable?
  - *Check*: Look for component reusability, DRY principles
  - *Severity*: Medium
 
- [ ] **UI-003**: UI component testing - Are unit tests written for UI components?
  - *Check*: Verify component test coverage
  - *Severity*: High
 
- [ ] **UI-004**: Monetary formatting - Are monetary amounts formatted properly?
  - *Check*: Verify currency formatting, decimal places
  - *Severity*: High
 
- [ ] **UI-005**: Monetary input shortcuts - Do input fields allow 'k' or 'm' notation?
  - *Check*: Verify 5k becomes 5,000, 2m becomes 2,000,000
  - *Severity*: Low
 
- [ ] **UI-006**: Effective date default - Is Effective/Start date defaulted to Today?
  - *Check*: Verify date field defaults (unless specified in BRD)
  - *Severity*: Medium
 
- [ ] **UI-007**: Expiration date default - Is Expiration/End date defaulted to 1 year from Today?
  - *Check*: Verify date field defaults (unless specified in BRD)
  - *Severity*: Medium
 
---
 
## 4. Pull Request Specifics
 
### 4.1 PR Metadata and Process
**Priority**: HIGH | **Automated**: High
 
- [ ] **PR-001**: PR description - Does PR description clearly explain the changes?
  - *Check*: Verify description is comprehensive and clear
  - *Severity*: High
 
- [ ] **PR-002**: Issue references - Are related issues/tickets referenced?
  - *Check*: Look for #123, Fixes #456 in description
  - *Severity*: High
 
- [ ] **PR-003**: Scope adherence - Are changes limited to ticket scope?
  - *Check*: Identify scope creep, unrelated changes
  - *Severity*: High
 
- [ ] **PR-004**: Unnecessary changes - Are there unnecessary or unrelated changes?
  - *Check*: Look for formatting-only changes, commented code, debugging statements
  - *Severity*: Medium
 
- [ ] **PR-005**: Branch currency - Is branch up to date with target branch?
  - *Check*: Verify no merge conflicts, recent sync
  - *Severity*: Critical
 
- [ ] **PR-006**: Build status - Does the code build successfully?
  - *Check*: Verify build pipeline passes
  - *Severity*: Critical
 
- [ ] **PR-007**: CI/CD checks - Do all pipeline checks pass?
  - *Check*: Verify tests, linting, security scans pass
  - *Severity*: Critical
 
- [ ] **PR-008**: Branch deletion - Will branch auto-delete after merge?
  - *Check*: Verify branch cleanup settings
  - *Severity*: Low
 
---
 
## Agent Implementation Guidelines
 
### Git Commands for Different Review Types
 
**Use these Git commands to gather code for review based on review type:**
 
#### For Last N Commits Review:
```bash
# List last N commits
git log -N --oneline
 
# Get detailed commit info
git log -N --stat --pretty=format:"%h - %an, %ad : %s" --date=short
 
# Show changes in last N commits
git diff HEAD~N..HEAD
 
# Get changed files in last N commits
git diff HEAD~N..HEAD --name-status
 
# Show detailed diff for each commit
git show <commit-hash>
```
 
#### For Branch Difference Review:
```bash
# Show current branch
git branch --show-current
 
# List commits in current branch not in Dev/Main
git log Dev/Main..HEAD --oneline
 
# Show file statistics for branch differences
git diff Dev/Main...HEAD --stat
 
# Show detailed code differences
git diff Dev/Main...HEAD
 
# Show only changed file names
git diff Dev/Main...HEAD --name-only
 
# Show changed files with status
git diff Dev/Main...HEAD --name-status
 
# Find merge base (common ancestor)
git merge-base Dev/Main HEAD
 
# Compare with master branch
git diff master...HEAD --stat
```
 
#### For Full Codebase Review:
```bash
# List all solution files
Get-ChildItem -Recurse -Include *.cs,*.csproj,*.json,*.yaml
 
# Get git status
git status
 
# Check for uncommitted changes
git diff
 
# Get current branch and state
git branch -v
```
 
### Handling Git Command Auto-Approval
 
**Implementation Strategy:**
 
When the user selects auto-approval mode:
1. **Batch Execution**: Execute all necessary git commands in sequence without pausing
2. **Transparent Logging**: Log each command before execution with "🚀 Auto-executing: `git ...`"
3. **Error Handling**: If any git command fails, log the error and continue with review using available data
4. **Session-Only**: Auto-approval preference applies only to current review session
 
When the user selects manual approval (or default):
1. **Command Preview**: Before each git command, show:
   - What the command does
   - Why it's needed for the review
   - Expected output type
2. **Wait for Confirmation**: Don't execute until user approves (or provide skip option)
3. **Alternative Path**: If user skips a command, note it in the final report
 
**Example Implementation Pattern:**
 
```python
# Pseudo-code for auto-approval logic
if autoApproveGitCommands:
    log("🚀 Auto-approve enabled: Executing git commands automatically")
    commands = get_required_git_commands_for_review_type(review_type)
    for cmd in commands:
        log(f"🚀 Auto-executing: `{cmd}`")
        result = execute_git_command(cmd)
        if result.error:
            log(f"⚠️ Command failed: {result.error}, continuing with available data")
else:
    log("⏸️ Manual approval mode: You'll be asked to confirm each git command")
    commands = get_required_git_commands_for_review_type(review_type)
    for cmd in commands:
        explain_command(cmd)
        if ask_user_approval(cmd):
            result = execute_git_command(cmd)
        else:
            log(f"⏭️ Skipped: `{cmd}`")
```
 
**Safety Considerations:**
- Auto-approval only applies to READ-ONLY git commands (log, diff, show, branch, status, merge-base)
- NEVER auto-approve commands that modify the repository (commit, push, merge, rebase, checkout)
- If an unexpected command is needed, always ask for explicit approval
- Include auto-approval status in review metadata for audit trail
 
### Review Process Flow
1. **Determine Review Scope**
   - Ask user for review type (Full, Last N Commits, Branch Difference, Custom)
   - **MANDATORY**: Based on selection, ask required follow-up questions:
     - **Last N Commits**: Ask for commit count (default: 5)
     - **Branch Difference**: Ask for target branch (default: origin/Dev/Main) ⚠️ **REQUIRED**
     - **Custom Review**: Ask for files/folders to review
   - Document which Git commands will be needed
 
2. **Git Command Auto-Approval Preference**
   - Ask user if they want to enable auto-approval for git commands
   - If enabled: Set `autoApproveGitCommands = true` and proceed automatically
   - If disabled: Set `autoApproveGitCommands = false` and ask before each command
   - Default to manual approval if unclear
   - Log the chosen preference for transparency
 
3. **Execute Git Commands & Load Context**
   - **CHECKPOINT**: Verify all mandatory prompts from Step 1 are completed before proceeding
   - Execute appropriate Git commands based on approval setting
   - For auto-approve: Run all git commands automatically (git log, git diff, git branch, git merge-base, etc.)
   - For manual: Explain and wait for approval before each git command
   - Load relevant code sections based on git results
   - Identify files to review based on scope
 
4. **Initialization**
   - Fetch file details and changed sections
   - Load previous review comments (if applicable)
   - Initialize checklist state
   - Set review context (commit range, branch comparison, etc.)
 
5. **Static Analysis**
   - Run automated checks (linting, security scanning)
   - Parse SonarCloud/other tool outputs
   - Calculate code metrics (complexity, coverage)
 
6. **Code Inspection**
   - Parse C# code using available tools
   - Walk code structure to identify patterns
   - Apply category-specific checklist rules
   - Focus on changed sections for commit/branch reviews
 
7. **Scoring & Classification**
   - Track all issues by severity (Critical, High, Medium, Low)
   - Calculate point deductions: Critical (-5), High (-3), Medium (-1), Low (-0.5)
   - Calculate category scores: Code Quality, .NET Best Practices, Testing, Security, Performance, Documentation
   - Generate weighted overall score
   - Determine approval status based on score thresholds
 
8. **Report Generation**
   - Apply appropriate review template (Full, Commits, Branch, Custom)
   - Group findings by severity and category
   - Provide code references with file paths and line numbers
   - Include actionable recommendations with code examples
   - Add review metadata (scope, date, branch, commits, auto-approval setting)
   - Generate comprehensive score breakdown
 
9. **Deliver Results**
   - Present formatted review report
   - Highlight critical blocking issues
   - Provide clear action items prioritized by sprint
   - Include approval recommendation with conditions
 
### Severity Levels
- **Critical**: Must be fixed before merge (security, correctness, breaking changes)
- **High**: Should be fixed before merge (performance, testability, maintainability)
- **Medium**: Should be addressed soon (code quality, documentation)
- **Low**: Nice to have (style preferences, minor optimizations)
 
### Automation Strategy
- **Fully Automated**: Linting, formatting, static analysis, build/test status
- **Partially Automated**: Complexity metrics, duplication detection, security scanning
- **Manual Review**: Business logic correctness, architecture decisions, requirement alignment
 
### Integration Points
- **Version Control**: GitHub/Azure DevOps PR API
- **Static Analysis**: SonarCloud, Roslyn analyzers
- **Build System**: Azure Pipelines, GitHub Actions
- **Logging**: Application Insights, structured logs
- **Security**: Dependency scanning, SAST tools
 
### Agent Configuration
```json
{
  "reviewConfig": {
    "minimumCoverageThreshold": 80,
    "maxCyclomaticComplexity": 10,
    "maxMethodLength": 50,
    "maxClassLength": 500,
    "blockingCategories": ["Security", "Error Handling", "Testing"],
    "autoApproveIfScoreAbove": 95,
    "requireManualReviewIfScoreBelow": 70
  }
}
```
 
### Standard Review Output Template
 
**ALL reviews MUST include the following sections:**
 
```markdown
# 📊 Code Review Report
 
## Review Metadata
- **Review Type**: [Full Codebase | Last N Commits | Branch Difference | Custom]
- **Review Scope**: [Description of what was reviewed]
- **Git Auto-Approval**: [✅ Enabled | ❌ Manual Approval]
- **Date**: [YYYY-MM-DD HH:MM]
- **Branch**: [Current branch name]
- **Compared With**: [Base branch, if applicable]
- **Commits Analyzed**: [Commit range or "N/A" for full review]
- **Files Reviewed**: [Count]
- **Lines of Code Reviewed**: [Approximate count]
 
---
 
## 🎯 Overall Assessment
 
**Status**: ✅ Approved | ✅ Approved with Recommendations | ⚠️ Needs Changes | ❌ Rejected
 
**Overall Score**: XX/100 (Grade: A+ | A | B+ | B | C | D | F)
 
### Score Breakdown by Category
 
| Category | Score | Weight | Status |
|----------|-------|--------|--------|
| **General Code Quality** | XX/100 | 25% | ✅/⚠️/❌ |
| **C# & .NET Best Practices** | XX/100 | 20% | ✅/⚠️/❌ |
| **Testing** | XX/100 | 15% | ✅/⚠️/❌ |
| **Security** | XX/100 | 25% | ✅/⚠️/❌ |
| **Performance** | XX/100 | 10% | ✅/⚠️/❌ |
| **Documentation** | XX/100 | 5% | ✅/⚠️/❌ |
| **Total Weighted Score** | **XX/100** | 100% | [Status] |
 
### Issue Summary
 
| Severity | Count | Points Deducted |
|----------|-------|-----------------|
| 🔴 Critical | X | -XX |
| 🟠 High Priority | X | -XX |
| 🟡 Medium Priority | X | -XX |
| 🟢 Low Priority | X | -XX |
| **Total Issues** | **XX** | **-XX points** |
 
---
 
## 🔴 Critical Issues (X) - MUST FIX BEFORE MERGE
 
### ❌ **[CHECK-ID]**: [Issue Title]
**Severity**: CRITICAL  
**File**: [path/to/file.cs](path/to/file.cs#LXX)  
**Category**: [Security | Error Handling | Correctness]
 
**Issue**: [Detailed description of the problem]
 
**Code**:
```csharp
// Problem code
```
 
**Fix**:
```csharp
// Corrected code
```
 
**Impact**: [What happens if not fixed]
 
---
 
## 🟠 High Priority Issues (X) - SHOULD FIX BEFORE MERGE
 
[Same format as Critical Issues]
 
---
 
## 🟡 Medium Priority Issues (X) - ADDRESS SOON
 
[Same format, condensed]
 
---
 
## 🟢 Low Priority Issues (X) - NICE TO HAVE
 
[Same format, condensed]
 
---
 
## ✅ What's Done Well
 
[List positive findings - good patterns, excellent security, etc.]
 
---
 
## 📋 Action Items
 
### Priority 1 (This Sprint - Blocking):
1. ✋ [Critical issue 1]
2. ✋ [Critical issue 2]
 
### Priority 2 (Next Sprint):
1. 🔄 [High priority issue 1]
2. 🔄 [High priority issue 2]
 
### Priority 3 (Backlog):
1. 📌 [Medium priority issue 1]
 
---
 
## 💡 Recommendations
 
[Strategic recommendations for architecture, patterns, future improvements]
 
---
 
## 🎯 Approval Decision
 
**Recommendation**: ✅ APPROVE | ⚠️ APPROVE WITH CONDITIONS | ❌ REQUEST CHANGES
 
**Conditions for Approval** (if applicable):
- [ ] Fix all critical issues
- [ ] Address X out of Y high priority issues
- [ ] Add unit tests for new logic
- [ ] Update documentation
 
**Next Steps**:
1. [Action 1]
2. [Action 2]
3. Re-review after fixes applied
 
---
 
**Reviewed by**: Code Review Agent v2.0  
**Review Completed**: [Timestamp]
```
 
### Review Type-Specific Templates
 
#### Template 1: Last N Commits Review
```markdown
## 📝 Commits Analyzed (X commits)
 
| # | Commit | Date | Author | Message | Files | Status |
|---|--------|------|--------|---------|-------|--------|
| 1 | `abc1234` | Mar 5 | Author | Message | 3 | ⚠️ Issues |
| 2 | `def5678` | Mar 4 | Author | Message | 5 | ✅ Good |
| ... |
 
### Commit-by-Commit Analysis
 
#### Commit 1: `abc1234` - [Message]
**Date**: March 5, 2026  
**Files Changed**: [List]  
**Issues Found**: X Critical, Y High, Z Medium  
**Score**: XX/100
 
[Detailed issues for this commit]
 
---
```
 
#### Template 2: Branch Difference Review
```markdown
## 🔀 Branch Comparison
 
**Current Branch**: `Feature/XYZ`  
**Base Branch**: `Dev/Main`  
**Commits Ahead**: X  
**Commits Behind**: Y  
**Divergence Point**: `commit-hash`
 
### Changed Files
 
| File | Status | Lines Changed | Issues | Score |
|------|--------|---------------|--------|-------|
| [file1.cs](path) | Modified | +50, -10 | 2 | 85/100 |
| [file2.cs](path) | Added | +200 | 5 | 70/100 |
 
### Integration Risks
- ⚠️ [Risk 1: potential conflict with Dev/Main]
- ⚠️ [Risk 2: dependency changes]
 
### Merge Recommendation
✅ Safe to merge | ⚠️ Merge with caution | ❌ Do not merge
 
**Reason**: [Explanation]
```
 
---
 
---
 
## Maintenance and Updates
- Review checklist quarterly for relevance
- Update based on team feedback and emerging practices
- Incorporate new security vulnerabilities and patterns
- Align with updated .NET versions and framework changes
 
## References
- [Microsoft C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [.NET Architecture Guides](https://dotnet.microsoft.com/learn/dotnet/architecture-guides)
- [Clean Code Principles](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
 
---
 
## Changelog
- **v2.0** - March 5, 2026: Added interactive review scope selection, comprehensive scoring system, and review type-specific templates
- **v1.1** - February 27, 2026: Updated with solution folder structure organization
- **v1.0** - Initial specification document
 
*Last Updated: March 5, 2026*