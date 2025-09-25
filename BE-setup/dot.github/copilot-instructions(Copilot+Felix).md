---
applyTo: "**"
---

# Instructions:

This mode is designed to assist with planning and organizing feature documentation. The workflow consists of creating
structured documentation before development, maintaining a documentation system, and ensuring consistency across all
planning documents.

## MarketIQ Backend Development Guide

This guide provides essential knowledge for AI agents working with the MarketIQ Backend, a Go-based enterprise web service built on Beego framework that serves the CBRE commercial real estate data platform.

## Development Context

Important considerations for the development process include:

- Ensuring all new features are accompanied by appropriate documentation.
- Maintaining a clear separation between development and production environments.
- Implementing thorough testing procedures to validate new functionality.
- Ensuring best practices for code quality and maintainability, considering factors such as language idioms and design patterns.

## Project Context

## Documentation Structure

### Directory Organization

All documentation must be stored in the `docs` directory. The structure is maintained in `docs/tree.md`.

#### Directory Naming Convention

- **Features**: `feat-<feature-name>` (e.g., `feat-user-authentication`, `feat-user-settings-api`)
- **Fixes**: `fix-<issue-name>` (e.g., `fix-login-error`, `fix-auth-password`)

#### File Naming Convention

Within each feature/fix directory:

- PRD Document: `<directory-name>.prd.md`
- Plan Document: `<directory-name>.plan.md`

Example structure:

```
docs/
├── tree.md
├── features/
│   ├── feat-user-settings-api/
│   │   ├── feat-user-settings-api.prd.md
│   │   └── feat-user-settings-api.plan.md
│   ├── feat-search-api/
│   │   ├── feat-search-api.prd.md
│   │   └── feat-search-api.plan.md
├── fixes/
│   ├── fix-auth-password/
│   │   ├── fix-auth-password.prd.md
│   │   └── fix-auth-password.plan.md
```

### Initial Setup

1. If `docs` directory doesn't exist, create it
2. If `docs/tree.md` doesn't exist, create it with initial structure
3. Always read `docs/tree.md` first to understand existing documentation

## Workflow Phases

### Phase 1: PRD Document Creation

When asked to create a new feature or fix:

1. **Understand Requirements**

   - Gather all user requirements through clarifying questions
   - Identify the problem being solved
   - Define success criteria
   - Consider existing project architecture and patterns

2. **Create PRD Document**

   - Create appropriate directory: `docs/feat-<name>` or `docs/fix-<name>`
   - Create PRD file: `<directory-name>.prd.md`
   - Include the following sections:
     - **Title**: Feature/Fix name
     - **Overview**: Brief description
     - **Problem Statement**: What problem this solves
     - **User Stories**: Who benefits and how
     - **Functional Requirements**: Detailed feature specifications
     - **Non-Functional Requirements**: Performance, security, scalability
     - **API Specifications**: Endpoints, DTOs, response formats (if applicable)
     - **UI/UX Requirements**: Interface design, user flows (if applicable)
     - **Success Criteria**: How to measure completion
     - **Constraints & Assumptions**: Technical or business limitations
     - **Dependencies**: External systems or features required
     - **Security Considerations**: Authentication, authorization, data protection

3. **Update Tree Documentation**
   - Add new directory to `docs/tree.md`
   - Include brief description of the feature/fix
   - Maintain alphabetical ordering within categories

### Phase 2: Plan Document Creation

After PRD approval:

1. **Create Plan Document**

   - Create file: `<directory-name>.plan.md`
   - Structure with milestones and tasks
   - Consider project-specific patterns and existing codebase

2. **Plan Document Structure**

   ```markdown
   # Development Plan: [Feature Name]

   ## Overview

   Brief summary linking to PRD

   ## Milestones

   ### Milestone 1: [Name]

   - [ ] Status: Not Started
   - Description: What this milestone achieves
   - Estimated Duration: X days

   #### Tasks:

   - [ ] Task 1.1: Description
   - [ ] Task 1.2: Description
   - [ ] Task 1.3: Description

   ### Milestone 2: [Name]

   - [ ] Status: Not Started ...

   ## Technical Considerations

   - Architecture decisions
   - Technology choices
   - Integration points
   - Database schema changes
   - API contract definitions

   ## Testing Strategy

   - Unit tests required
   - Integration tests
   - User acceptance criteria
   - Performance benchmarks

   ## Risk Assessment

   - Potential blockers
   - Mitigation strategies
   - Rollback procedures
   ```

3. **Task Definition Guidelines**
   - Each milestone should represent a significant deliverable
   - Tasks should be atomic and completable in 1-4 hours
   - Include clear acceptance criteria for each task
   - Consider dependencies between tasks
   - Follow existing project patterns and standards

## Architecture Overview

**MarketIQ Backend** is a mission-critical Go web service (v1.23.0+) using:
- **Framework**: Beego v2.3.8 (REST API with embedded controllers)
- **Database**: PostgreSQL with Beego ORM (`miq` schema)
- **Cache**: Redis for performance optimization
- **Authentication**: OAuth 2.0 with JWT tokens
- **External Integration**: GraphQL mutations via EDP (External Data Provider)
- **Monitoring**: DataDog integration for prod/QA environments
- **Automation**: Database-driven entity synchronization framework

## Essential Development Patterns

### 1. Beego Controller Pattern

**Every controller must follow this exact error handling pattern:**

```go
// Standard controller structure
type MyController struct {
    web.Controller
}

func (c *MyController) Get() {
    ctxLog := logger.New(c.Ctx)
    
    // 1. Input validation
    if checkAndServeError(err, c.Controller, e.InvalidInputValue) {
        return
    }
    
    // 2. User authentication 
    u, err := user.FindByEmail(security.ExtractFromToken(c.Ctx.Request, security.EmailAddress))
    if checkAndServeError(err, c.Controller, e.InvalidInputValue) {
        ctxLog.Println(e.MissingUser, err)
        return
    }
    
    // 3. Business logic with error handling
    data, err := models.GetData()
    if checkAndServeError(err, c.Controller, e.GenericError) {
        return
    }
    
    // 4. Always use logAndServeJSON for responses
    logAndServeJSON(&data, c.Controller)
}
```

**Critical**: Never use `c.ServeJSON()` directly - always use `logAndServeJSON()` for audit logging.

### 2. Database Operations with miqorm

All database operations use the custom `miqorm` wrapper:

```go
// Always get ORM instance this way
o := miqorm.NewOrm()

// Transaction pattern for writes
ormobj := miqorm.NewOrm()
ormobj.Begin()
defer func() {
    if err != nil {
        ormobj.Rollback()
    } else {
        ormobj.Commit()
    }
}()
```

**Schema Convention**: All application tables use `miq.` prefix (e.g., `miq.miq_user`, `miq.miq_automation_rules`).

### 3. Write Operations Pattern

**All write operations trigger automation framework:**

```go
// 1. Perform database update first
result, err := models.UpdateEntity(data)
if checkAndServeError(err, c.Controller, e.GenericError) {
    return
}

// 2. Trigger automation (never fail main operation)
go func() {
    err := automationintegrator.TriggerAutomation(
        "updateProperty",    // method name
        "property",          // entity type
        propertyID,          // entity ID
        propertyData,        // changed data
        userContext,         // user context
    )
    if err != nil {
        logger.CommonLog.Errorf("Automation failed: %v", err)
    }
}()

// 3. Return immediately to user
logAndServeJSON(&result, c.Controller)
```

### 4. User Context Pattern

**Always pass user context through call chains:**

```go
// Extract from token
email := security.ExtractFromToken(c.Ctx.Request, security.EmailAddress)
u, err := user.FindByEmail(email)

// Pass through models
result, err := models.ProcessData(data, u, ctxLog)
```

## Critical Build & Quality Process

### Build Commands (MANDATORY)

```bash
# NEVER use 'go build' directly
./build    # Always use this script

# Before any build - MANDATORY code quality check
codacy-cli analyze --file path/to/changed/files.go
```

**Quality Gate**: Codacy analysis is required before every build. Address all critical/high-priority issues.

### Environment Setup

**Vault Token Required**: Get `vault-token-dev-cluster` from Jenkins or contact EDPDigiTechDevOpsTeam@cbre.com

```bash
# First time setup
env VAULT_TOKEN=<token> ./run

# Get permanent secrets (one-time)
env VAULT_TOKEN=<token> SHOW_SECRETS=on ./run
```

## Key Integration Points

### GraphQL Mutations (Write Operations)

All data writes go through EDP GraphQL:

```go
// Build mutation using patterns from models/write/
mutation := ingestion.NewUpdate().
    Entity("property").
    ID(propertyID).
    Fields(propertyData)

// Submit asynchronously 
err = api.SubmitMutation(mutation, userContext)
```

### Automation Framework

**Database-driven rules** in `miq.miq_automation_rules` automatically sync related entities:

- **Triggers**: Property updates → Availability updates
- **Rules**: JSON-based field mappings and conditions  
- **Processing**: Async workers with retry logic
- **Integration**: GraphQL + Kafka publishing

### Redis Caching

**Essential patterns:**

```go
// Use Redis for frequently accessed data
cacheKey := fmt.Sprintf("user:%d:settings", userID)
if cached := redis.Get(cacheKey); cached != nil {
    // Use cached data
}
// Otherwise fetch from DB and cache
```

## Environment-Specific Behaviors

**Local/Dev**: 
- `orm.Debug = true` (SQL logging enabled)
- JWT validation disabled
- Direct database access

**QA/Prod**:
- DataDog tracing enabled  
- All middleware active
- Vault-based secrets
- Performance monitoring

## Error Handling Standards

**Use these specific error types:**

```go
// Validation errors
if checkAndServeError(err, c.Controller, e.InvalidInputValue) {
    return
}

// Missing data
if checkAndServeError(err, c.Controller, e.MissingInputValue) {
    return  
}

// Access control
if !u.In(miqutils.AdminUser) {
    checkAndServeErrorWithHTTPCode(e.New(e.InsufficientPrivileges), 
        c.Controller, e.AccessDenied, http.StatusForbidden)
    return
}

// Generic server errors
if checkAndServeError(err, c.Controller, e.GenericError) {
    return
}
```

## Route Configuration

Routes defined in `routers/{entity}_router.go`:

```go
web.Router("/users/:id", &controllers.UserController{})
web.Router("/search", &controllers.SearchController{})
```

## Testing Approach

```bash
# Code coverage
go test ./... -coverprofile=c.out
go tool cover -html=c.out

# Integration tests with real database
go test -tags integration ./models/...
```

## Documentation Standards

### PRD Standards in `docs/`

- Use clear, concise language
- Include mockups or diagrams where helpful (using Mermaid syntax)
- Reference existing project components and patterns
- Consider authentication and authorization requirements
- Define clear API contracts with TypeScript interfaces
- Specify validation rules and error handling approaches
- Features: `feat-<name>/{name}.prd.md`
- Fixes: `fix-<name>/{name}.prd.md`
- Update `docs/tree.md` for organization

### Plan Standards in `docs/`

- Break down work into logical, testable milestones
- Consider both backend (GoLang) and frontend (Angular) tasks
- Include database migration requirements
- Specify integration points with existing features
- Account for testing and documentation time
- Features: `{name}.plan.md`  
- Fixes: `{name}.plan.md`
- Update `docs/tree.md` for organization

### Version Control

- Commit documentation changes with descriptive messages:
  - `docs: Add PRD for <feature-name>`
  - `docs: Update plan for <feature-name>`
  - `docs: Update tree.md with <feature-name>`

## Quality Checks

Before completing any phase:

- Ensure all required sections are present
- Verify naming conventions are followed
- Confirm tree.md is updated
- Check that documentation aligns with project standards
- Validate technical feasibility against existing architecture
- Ensure security and performance considerations are addressed

## Command Recognition

When user says:

- "Start a new feature for X" → Begin Phase 1 (PRD Creation)
- "Create the plan" → Begin Phase 2 (Plan Document)
- "Update the tree" → Update docs/tree.md
- "Review existing features" → List and summarize docs directory

## Review Checkpoints

- After PRD creation: Pause for user review before proceeding
- After Plan creation: Pause for user review before marking complete
- Always confirm directory and file names before creation
- Summarize key decisions for user validation

## Security Requirements

- **Authentication**: OAuth middleware validates all requests
- **Authorization**: Role-based access (`u.In(miqutils.AdminUser)`)
- **Audit Logging**: All operations logged via `logAndServeJSON`
- **Input Sanitization**: Middleware prevents XSS/injection

## Performance Considerations

- **Connection Pooling**: Database connections managed by ORM
- **Async Processing**: Background workers for heavy operations
- **Redis Caching**: Cache frequently accessed lookups  
- **GraphQL Batching**: Batch related mutations where possible

## Common Pitfalls

❌ **Never:**
- Use `c.ServeJSON()` directly (breaks audit logging)
- Skip Codacy analysis before builds
- Block write operations on automation processing

✅ **Always:**
- Follow the controller error handling pattern
- Use `logAndServeJSON()` for all responses
- Pass user context through call chains
- Check Codacy analysis results

Remember: This is a mission-critical enterprise system. Always prioritize data integrity, proper error handling, and comprehensive logging. When in doubt, follow the established patterns in existing working code.