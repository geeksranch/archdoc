# Contributing Guidelines

## Overview

Thank you for contributing to the Enterprise Hybrid Architecture Documentation. This document provides guidelines for updating and maintaining the architecture documentation.

## Documentation Standards

### File Naming Conventions

- Use lowercase letters and hyphens for file names (e.g., `business-context.md`)
- Each phase directory should contain a `README.md` as the entry point
- Use descriptive names that reflect the content

### Markdown Formatting

1. **Headers**: Use proper heading hierarchy (H1 for title, H2 for sections, H3 for subsections)
2. **Links**: Use relative links for internal documentation
3. **Tables**: Use markdown tables for structured information
4. **Code Blocks**: Use fenced code blocks with language identifiers

### Mermaid Diagrams

All architecture diagrams should use Mermaid syntax for version control and maintainability:

```markdown
\`\`\`mermaid
graph TD
    A[Component A] --> B[Component B]
\`\`\`
```

Supported diagram types:
- `graph` / `flowchart` - Flow diagrams
- `C4Context` - C4 Context diagrams
- `mindmap` - Mind maps
- `quadrantChart` - Quadrant charts
- `gantt` - Gantt charts
- `pie` - Pie charts
- `stateDiagram-v2` - State diagrams
- `block-beta` - Block diagrams

## Contribution Process

### 1. Creating Changes

1. Clone the repository
2. Create a new branch from `main`:
   ```bash
   git checkout -b feature/your-change-description
   ```
3. Make your changes following the documentation standards
4. Test Mermaid diagrams using the [Mermaid Live Editor](https://mermaid.live/)

### 2. Architecture Decision Records (ADR)

For significant architecture changes:

1. Copy the [ADR template](templates/architecture-decision-record.md)
2. Fill in all required sections
3. Submit for review by the Architecture Review Board (ARB)

### 3. Change Requests

For changes to existing architecture:

1. Complete the [Change Request template](templates/change-request-template.md)
2. Submit through the change management process
3. Wait for CAB approval before implementation

### 4. Pull Request Guidelines

- Provide a clear description of changes
- Reference related issues or ADRs
- Ensure all Mermaid diagrams render correctly
- Update navigation links if adding new files
- Update the artifact summary table in main README if applicable

## Directory Structure

```
├── README.md                          # Main overview and navigation
├── docs/
│   ├── phase-a-architecture-vision/   # Phase A documentation
│   ├── phase-b-business-architecture/ # Phase B documentation
│   ├── phase-c-information-systems/   # Phase C documentation
│   ├── phase-d-technology-architecture/ # Phase D documentation
│   ├── phase-e-opportunities-solutions/ # Phase E documentation
│   ├── phase-f-migration-planning/    # Phase F documentation
│   ├── phase-g-implementation-governance/ # Phase G documentation
│   ├── phase-h-change-management/     # Phase H documentation
│   └── cross-cutting/                 # Cross-cutting concerns
├── templates/                         # Document templates
├── DOC/                              # Existing documentation
└── CONTRIBUTING.md                    # This file
```

## Review Process

### Documentation Reviews

- All changes require at least one reviewer
- Architecture changes require ARB review
- Security-related changes require Security team review

### Approval Matrix

| Change Type | Reviewers Required |
|-------------|-------------------|
| Minor updates (typos, formatting) | 1 team member |
| Content updates | 2 team members |
| New architecture patterns | ARB approval |
| Security architecture changes | Security team + ARB |
| Cross-cutting concerns | Enterprise Architecture team |

## Contact

- **Enterprise Architecture Team**: ea-team@example.com
- **Architecture Review Board**: arb@example.com
- **Security Team**: security@example.com

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2024-01-01 | EA Team | Initial documentation structure |

---

*Thank you for helping maintain high-quality architecture documentation!*
