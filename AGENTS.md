# Project Development Guidelines

<!-- TODO: Add a brief description of your project here -->
<!-- Example: A REST API for managing user authentication and authorization -->

This project uses Kiro Best Practices boilerplate for automated quality checks and development standards.

## Project-specific information

<!-- TODO: Fill in the sections below with your project details -->

### Architecture overview
<!-- Describe your project's architecture, key components, and how they interact -->

### Build and run
<!-- Add commands to build and run your project -->
```bash
# Example:
# npm install
# npm run dev
# npm run build
```

### Testing
<!-- Add your project-specific test commands -->
```bash
# Example:
# npm test
# npm run test:e2e
# npm run test:integration
```

### Deployment
<!-- Describe how to deploy your project -->
```bash
# Example:
# npm run deploy:staging
# npm run deploy:production
```

### Environment variables
<!-- List required environment variables -->
```bash
# Example:
# DATABASE_URL=postgresql://...
# API_KEY=your-api-key
# NODE_ENV=development|production
```

### Project-specific conventions
<!-- Add any project-specific coding conventions, patterns, or requirements -->
<!-- Example:
- All API endpoints must include rate limiting
- Database migrations must be reversible
- Feature flags must be documented in docs/features.md
-->

---

## How steering and hooks work

### Steering documents (`.kiro/steering/`)
- Automatically guide all AI interactions with best practices
- Active immediately, no restart needed
- Cover languages (TypeScript, Python, Go), cloud (AWS, Azure, GCP), infrastructure (Terraform, CDK, Docker)
- See: https://docs.kiro.ai/steering

### Hooks (`.kiro/hooks/`)
- **Automatic hooks**: Trigger on file saves (testing, linting, security scans)
- **Manual hooks**: Available via Agent Hooks panel (commit helpers, dependency updates, coverage checks)
- Require Kiro restart after installation to activate
- See: https://docs.kiro.ai/hooks

---

## Code style guidelines

The steering documents enforce these standards automatically:

- **TypeScript**: Strict mode, explicit return types, PascalCase for classes, camelCase for variables
- **Python**: PEP 8, type hints, virtual environments, format with `black`
- **Go**: Use `gofmt`, explicit error handling, table-driven tests
- **Infrastructure**: Terraform modules with remote state, CDK with organized structure, Docker multi-stage builds
- **Security**: No hardcoded secrets, input validation, least privilege access
- **Git**: Conventional commits (`feat:`, `fix:`, `docs:`, etc.)

## Testing best practices

Always run tests with minimal verbosity to prevent session timeouts:

```bash
# TypeScript/JavaScript
npm test -- --silent
npx jest --silent

# Python
pytest -q
python -m pytest --tb=short -q

# Go
go test -v ./...
go test -run TestSpecific

# Coverage
npm test -- --coverage
pytest --cov
go test -cover ./...
```

## MCP server usage

If MCP servers are configured, use them for enhanced development:

### Context7 - Verify dependency compatibility
- Check library compatibility before adding dependencies
- Get up-to-date documentation for libraries
- Example: `mcp_Context7_resolve_library_id` then `mcp_Context7_get_library_docs`

### AWS Documentation - Access current AWS docs
- Search AWS documentation for best practices
- Read specific AWS service documentation
- Get recommendations for related AWS resources

### Configuration
- MCP servers configured in `.kiro/settings/mcp.json`
- See `mcp-best-practices.md` steering document for patterns
- Use `autoApprove` only for trusted, low-risk tools

## Cloud CLI best practices

### AWS CLI
- Always use `--no-cli-pager` to prevent interactive paging
- Use `--output json` for scripts, `--output table` for humans
- Use `--query` to filter results and reduce output

### Azure CLI
- Use `--output json|table|tsv` for different formats
- Set defaults with `az configure --defaults` to avoid repetition
- Use `--no-wait` for long-running operations

### GCP CLI (gcloud)
- Use `--format=json|table|csv` for output control
- Set defaults with `gcloud config set` for region/zone
- Use `--quiet` to suppress prompts in scripts

## Development workflow

### File management
- Never create duplicate files with suffixes like `_fixed`, `_clean`, `_backup`
- Work iteratively on existing files
- Let hooks handle quality checks automatically

### Dependency management
- Use latest stable versions of dependencies
- Verify compatibility before adding (use Context7 MCP if available)
- Security scans run automatically on package file changes
- Remove unused dependencies regularly with `go mod tidy`, `npm prune`, etc.

### Commit conventions
Use conventional commit format for clear history:
- `feat(scope): description` - New features
- `fix(scope): description` - Bug fixes
- `docs(scope): description` - Documentation changes
- `refactor(scope): description` - Code refactoring
- `test(scope): description` - Test additions/changes
- `chore(scope): description` - Maintenance tasks

### Security requirements
- Never hardcode secrets, API keys, or passwords
- Use environment variables for sensitive configuration
- Validate all user inputs
- Enable encryption for sensitive data at rest and in transit
- Follow least privilege principle for access control
- Environment file validation runs automatically

## Customizing for your project

### Enable/disable hooks
Edit `.kiro/hooks/*.kiro.hook` files to enable/disable specific hooks:
```json
{
  "enabled": false,  // Set to true to enable
  "name": "Hook Name"
}
```

### Add project-specific steering
Create custom steering documents in `.kiro/steering/`:
```markdown
---
title: Project Specific Standards
inclusion: always
---

# Project-Specific Guidelines
- Your team's coding standards
- Project architecture patterns
- Deployment procedures
```

### Adjust file patterns
Modify hook file patterns to match your project structure:
```json
{
  "when": {
    "type": "fileEdited",
    "patterns": [
      "src/**/*.ts",      // Your source directory
      "lib/**/*.js",      // Your library directory
      "**/*.custom"       // Your custom file types
    ]
  }
}
```

### Configure MCP servers
Add project-specific MCP servers in `.kiro/settings/mcp.json`:
```json
{
  "mcpServers": {
    "your-server": {
      "command": "uvx",
      "args": ["your-mcp-server@latest"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

## Available steering documents

The following best practice guides are included and automatically active:

- **Languages**: TypeScript, Python, Go
- **Cloud**: AWS CLI, Azure CLI, GCP CLI
- **Infrastructure**: Terraform, CDK, Docker
- **Frameworks**: React
- **Tools**: Git, MCP, Testing
- **Standards**: Development Standards, Security Best Practices

Each steering document provides language/tool-specific guidance that automatically influences AI interactions.

## References

- Kiro Steering: https://docs.kiro.ai/steering
- Kiro Hooks: https://docs.kiro.ai/hooks
- MCP Protocol: https://modelcontextprotocol.io/
- Conventional Commits: https://www.conventionalcommits.org/
- OWASP Security: https://owasp.org/www-project-top-ten/