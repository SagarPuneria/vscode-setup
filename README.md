# VSCode Development Setup

This repository contains preconfigured VSCode setups for both Backend (BE) and Frontend (FE) development environments, designed to streamline development workflows and ensure consistent development practices across teams.

## Project Structure

```
vscode-setup/
├── README.md                 # This file
├── settings.json            # Global VSCode settings configuration
├── BE-setup/                 # Backend development configuration
│   ├── docs/                 # Project documentation
│   │   ├── tree.md          # Documentation structure tracker
│   │   ├── features/        # Feature documentation (empty)
│   │   └── fixes/           # Fix documentation (empty)
│   ├── .codacy/             # Code quality configuration
│   │   └── codacy.yaml      # Codacy static analysis settings
│   ├── .github/             # GitHub workflow and instructions
│   │   ├── copilot-instructions.md  # AI-assisted development guidelines
│   │   └── instructions/
│   │       └── build.instructions.md # Build process documentation
│   └── .vscode/             # VSCode configuration
│       └── launch.json      # Debug configuration for Go
└── FE-setup/                # Frontend development configuration
    └── .vscode/             # VSCode configuration
        ├── launch.json      # Debug configuration for Angular
        └── tasks.json       # Build and serve tasks for Angular
```

## Backend Setup (BE-setup)

### Technology Stack
- **Primary Language**: Go (GoLang)
- **Project**: MarketIQ Backend
- **Code Quality**: Codacy integration with multiple language support

### Key Features

#### Development Environment
- **Go Debug Configuration**: Pre-configured launch settings for Go applications
- **Environment Variables**: Local development environment support
- **Code Quality**: Comprehensive Codacy configuration supporting multiple languages and tools

#### Supported Languages & Tools
- **Languages**: Dart, Go, Java, Node.js, Python
- **Analysis Tools**: 
  - dartanalyzer, ESLint, Lizard (complexity analysis)
  - PMD (code analysis), Pylint, Revive (Go linter)
  - Semgrep (static analysis), Trivy (security scanner)

#### Documentation System
- **Structured Documentation**: Feature and fix documentation workflow
- **PRD & Planning**: Product Requirements Documents and development planning
- **GitHub Copilot Integration**: AI-assisted development with custom instructions

#### Build Process
- **Custom Build Script**: Uses `./build` command instead of standard `go build`
- **Code Review Workflow**: Mandatory Codacy analysis before building
- **Quality Gates**: Automated code quality checks and security scanning

### Usage Instructions

1. **Setup Development Environment**:
   ```bash
   # Copy BE-setup/.vscode to your Go project root
   cp -r BE-setup/.vscode /path/to/your/go/project/
   
   # Copy documentation structure
   cp -r BE-setup/docs /path/to/your/go/project/
   
   # Copy Codacy configuration
   cp -r BE-setup/.codacy /path/to/your/go/project/
   ```

2. **Install Required Extensions**:
   - Go extension for VSCode
   - Codacy MCP extension
   - GitHub Copilot (optional but recommended)

3. **Development Workflow**:
   ```bash
   # 1. Make code changes
   # 2. Run Codacy analysis
   codacy-cli analyze --file path/to/changed/file.go
   # 3. Fix critical issues
   # 4. Build the project
   ./build
   # 5. Test and commit
   ```

## Frontend Setup (FE-setup)

### Technology Stack
- **Framework**: Angular
- **Development Server**: Angular CLI with SSL support
- **Domain**: local.miq.cbre.com (CBRE MarketIQ)

### Key Features

#### Development Server Configuration
- **SSL Support**: Pre-configured with local SSL certificates
- **Custom Domain**: Configured for `local.miq.cbre.com:443`
- **Proxy Configuration**: Built-in proxy support for API calls
- **Background Process**: Non-blocking development server

#### Debug Configuration
- **Chrome Integration**: Launch Chrome with SSL support
- **Port 443**: Configured for production-like HTTPS development
- **Pre-launch Tasks**: Automatically starts `ng serve` before debugging

#### Build Tasks
- **Dual Configuration**: Two different task configurations available
- **Memory Optimization**: Node.js memory allocation for large projects
- **Problem Matching**: TypeScript compilation error detection
- **Background Execution**: Non-blocking task execution

### Usage Instructions

1. **Setup Development Environment**:
   ```bash
   # Copy FE-setup/.vscode to your Angular project root
   cp -r FE-setup/.vscode /path/to/your/angular/project/
   ```

2. **SSL Certificate Setup**:
   ```bash
   # Ensure SSL certificates exist at:
   # app-configs/utils/ssl/local_miq_cbre_com.key
   # app-configs/utils/ssl/local_miq_cbre_com.cer
   ```

3. **Required Files**:
   - `proxy.conf.js` - Proxy configuration for API calls
   - SSL certificates in `app-configs/utils/ssl/`

4. **Development Workflow**:
   - Press `F5` or use Debug menu to start development server
   - Server will run on `https://local.miq.cbre.com:443`
   - Chrome will automatically launch and connect

## Installation Guide

### Prerequisites
- VSCode installed
- Node.js (for FE development)
- Go (for BE development)
- Git

### Quick Setup

1. **Clone/Download this repository**:
   ```bash
   git clone <repository-url>
   cd vscode-setup
   ```

2. **Apply Global Settings** (Recommended):
   ```bash
   # For macOS/Linux - Copy to VSCode user settings
   cp settings.json ~/Library/Application\ Support/Code/User/settings.json
   
   # For Windows - Copy to VSCode user settings
   cp settings.json %APPDATA%/Code/User/settings.json
   
   # Or manually merge settings through VSCode UI:
   # Code → Preferences → Settings → Open Settings (JSON)
   ```

3. **Choose your setup**:
   ```bash
   # For Backend development
   cd BE-setup
   
   # For Frontend development  
   cd FE-setup
   ```

4. **Copy configuration to your project**:
   ```bash
   # Backend
   cp -r BE-setup/{.vscode,.codacy,.github,docs} /path/to/your/go/project/
   
   # Frontend
   cp -r FE-setup/.vscode /path/to/your/angular/project/
   ```

## Configuration Details

### Global Settings Features
The included `settings.json` provides:
- **Multi-language support**: Optimized for Go, Python, TypeScript/JavaScript
- **AI Integration**: GitHub Copilot with custom instructions
- **Security Tools**: Snyk, Veracode, Cycode integration
- **Database Tools**: Database client with AI-powered queries
- **Git Enhancement**: Advanced GitLens features and auto-fetch
- **Code Quality**: Comprehensive linting and formatting rules

### Backend Configuration
- **Debug Port**: Uses Go's default debugging capabilities
- **Environment**: Local development with ENV=local
- **Code Quality**: Multi-language static analysis
- **Documentation**: Structured PRD and planning workflow

### Frontend Configuration
- **Development Port**: 443 (HTTPS)
- **SSL**: Required for local development
- **Memory**: Optimized for large Angular applications
- **Proxy**: Configured for backend API integration

## Best Practices

### Code Quality
- Always run Codacy analysis before building
- Address critical and high-priority issues
- Use the custom build scripts provided

### Documentation
- Follow the structured documentation approach
- Create PRDs before development
- Maintain the documentation tree

### Development Workflow
- Use the provided debug configurations
- Follow the pre-configured build processes
- Maintain consistent development environments

## Troubleshooting

### Common Issues

1. **SSL Certificate Errors**: Ensure certificates exist in the correct location
2. **Port 443 Access**: May require sudo permissions on macOS/Linux
3. **Codacy Tool Missing**: Install from VSCode Extensions Marketplace
4. **Go Debug Issues**: Ensure Go extension is properly installed
5. **Python Formatter Issues**: Install Black formatter: `pip install black`
6. **Settings Not Applied**: Ensure settings.json is in the correct VSCode user directory
7. **Security Tool Conflicts**: Configure trusted folders for Snyk and other security tools

### Required Extensions
To fully utilize the global settings, install these VSCode extensions:
- **Go** (golang.go)
- **Python** (ms-python.python)
- **Black Formatter** (ms-python.black-formatter)
- **Pylance** (ms-python.vscode-pylance)
- **GitHub Copilot** (github.copilot)
- **GitLens** (eamodio.gitlens)
- **Snyk Security** (snyk-security.snyk-vulnerability-scanner)
- **Database Client** (cweijan.vscode-database-client2)
- **Docker** (ms-azuretools.vscode-docker)
- **Atlassian for VS Code** (atlassian.atlascode)

### Support
- Check the respective `instructions/` directories for detailed setup guides
- Refer to the documentation in `docs/` for project-specific information
- Review the Copilot instructions for AI-assisted development guidelines

## Contributing

When adding new configurations:
1. Follow the existing directory structure
2. Update this README with new features
3. Include proper documentation
4. Test configurations before committing

---

*This setup is designed for enterprise-level development with emphasis on code quality, security, and maintainability.*