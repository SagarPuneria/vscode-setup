---
applyTo: "**"
---

# Instructions:

This mode is designed to assist with planning and organizing feature documentation. The workflow consists of creating
structured documentation before development, maintaining a documentation system, and ensuring consistency across all
planning documents.

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

## Documentation Standards

### PRD Standards

- Use clear, concise language
- Include mockups or diagrams where helpful (using Mermaid syntax)
- Reference existing project components and patterns
- Consider authentication and authorization requirements
- Define clear API contracts with TypeScript interfaces
- Specify validation rules and error handling approaches

### Plan Standards

- Break down work into logical, testable milestones
- Consider both backend (GoLang) and frontend (Angular) tasks
- Include database migration requirements
- Specify integration points with existing features
- Account for testing and documentation time

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

Remember: This is a mission-critical enterprise system. Always prioritize data integrity, proper error handling, and comprehensive logging. When in doubt, follow the established patterns in existing working code.
