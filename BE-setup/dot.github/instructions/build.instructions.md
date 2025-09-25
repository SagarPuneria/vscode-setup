---
applyTo: '**'
---
Coding standards, domain knowledge, and preferences that AI should follow.

## Prerequisites

### Codacy MCP Tool Installation
Before starting development, ensure the Codacy MCP tool is available:

1. **Check if Codacy MCP tool is installed**:
   ```bash
   # Test if Codacy MCP tool is available
   # If this command works, you're ready to proceed
   codacy-cli --version
   ```

2. **If Codacy MCP tool is NOT installed**:
   - Visit the **Microsoft VS Code Extensions Marketplace**
   - Search for "Codacy" or "Model Context Protocol (MCP)"
   - Install the official Codacy MCP extension
   - Alternatively, follow Codacy's official installation guide for MCP tools

## Build and Development Commands

### Building the Project
- **Always use `./build` command** to build the marketiq-backend project
- The `./build` script contains all necessary build commands and configurations
- **Do not use `go build` directly** - use `./build` instead as it handles project-specific requirements

### Code Quality and Review

#### Mandatory Code Review Process
- **Always run Codacy analysis** before building any code changes
- Use the Codacy MCP tool to perform static code analysis and quality checks
- Address all critical and high-priority issues identified by Codacy
- Code review helps maintain consistency and catches potential issues early

#### Codacy Analysis Commands
```bash
# For specific file analysis
codacy-cli analyze --file path/to/your/file.go

# For directory analysis  
codacy-cli analyze --file path/to/directory/

# For full project analysis
codacy-cli analyze
```

### Development Workflow
1. **Make code changes**
2. **Run Codacy analysis** to review code quality and identify issues:
   ```bash
   # Analyze the specific files you changed
   codacy-cli analyze --file path/to/changed/file.go
   ```
3. **Fix any critical or high-priority issues** found by Codacy
4. **Build the project** using the standard build command:
   ```bash
   ./build
   ```
5. **Test the changes** to ensure functionality works as expected
6. **Commit when build is successful** and code quality checks pass

### Build Process Details
The `./build` script performs the following:
- Compiles Go code with proper flags and optimizations
- Runs dependency management and module verification
- Validates project structure and configurations
- Generates necessary build artifacts
- Ensures cross-platform compatibility

### Important Notes
- **Code quality first**: Never skip the Codacy review step
- **Always use `./build`**: This is the standard and preferred way to build this project
- **Consistent environments**: Using `./build` ensures consistency across different development environments
- **Security and maintainability**: Codacy analysis ensures code maintainability, security, and best practices
- **Build validation**: The build process includes all necessary flags, dependencies, and build steps