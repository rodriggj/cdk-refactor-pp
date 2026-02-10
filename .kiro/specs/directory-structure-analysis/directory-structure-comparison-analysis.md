# Directory Structure Comparison Analysis

## Executive Summary

This document provides a comprehensive analysis of two AWS CDK projects:
1. **risbi-data-pipeline-from-snowflake-to-db2** (Python-based data pipeline)
2. **service-agreement-reconciliation-pre-processor** (TypeScript-based CDK template)

The analysis identifies structural differences and provides actionable recommendations for aligning the service-agreement project with the risbi-data-pipeline organizational structure.

---

## Part 1: Individual Project Analysis

### 1.1 risbi-data-pipeline-from-snowflake-to-db2

#### Project Overview
- **Purpose**: ETL data pipeline that extracts data from Snowflake, stages it in AWS S3, and loads it into an on-premises DB2 database
- **Technology Stack**: Python 3.9+, AWS CDK (Python), AWS Lambda, Snowflake, DB2
- **Deployment**: Multi-environment (Development, Staging, Production)
- **Architecture**: Three-stage pipeline with separate Lambda functions for each stage

#### Directory Structure

```
risbi-data-pipeline-from-snowflake-to-db2/
├── .git/                          # Git version control
├── .github/                       # GitHub configuration
│   ├── workflows/                 # CI/CD pipeline definitions
│   ├── CODEOWNERS                 # Code ownership rules
│   ├── dependabot.yml            # Dependency management
│   └── pull_request_template.md  # PR template
├── application/                   # Lambda function code (APPLICATION LAYER)
│   ├── extract-from-snowflake/   # Stage 1: Snowflake extraction
│   │   ├── code_lib/             # Shared utilities
│   │   ├── Dockerfile            # Container definition
│   │   ├── requirements.txt      # Python dependencies
│   │   ├── snowflake_to_s3_lambda.py  # Main Lambda handler
│   │   └── *.sql                 # SQL query files
│   ├── ftp-to-onprem/           # Stage 2: FTP transfer
│   │   ├── Dockerfile
│   │   ├── requirements.txt
│   │   └── ftp_to_onprem_lambda.py
│   └── load-into-db2/           # Stage 3: DB2 loading
│       ├── certs/               # SSL certificates
│       ├── so_files/            # Shared object libraries
│       ├── Dockerfile
│       ├── requirements.txt
│       ├── db2.py               # DB2 connection utilities
│       └── s3_to_db2_lambda.py
├── diagram/                      # Architecture diagrams
│   ├── *.png                    # Diagram images
│   ├── *.py                     # Diagram generation scripts
│   ├── poetry.lock              # Poetry dependency lock
│   └── pyproject.toml           # Poetry configuration
├── infrastructure/               # CDK infrastructure code
│   ├── helpers/                 # Helper modules
│   │   ├── iam.py              # IAM utilities
│   │   └── kms.py              # KMS utilities
│   ├── stacks/                  # CDK stack definitions
│   │   ├── iam_policystatements.py
│   │   ├── main.py             # Main stack
│   │   └── tpa_infra.py        # TPA-specific stack
│   ├── app.py                   # CDK app entry point
│   ├── cdk.json                 # CDK configuration
│   ├── config.json              # Environment-specific config
│   └── requirements.txt         # Infrastructure dependencies
├── tests/                        # Test directory
│   └── __init__.py
├── .changerc                     # Change management config
├── .gitignore                    # Git ignore rules
├── catalog-info.yaml            # Backstage catalog metadata
├── CONTRIBUTING.md              # Contribution guidelines
├── NETWORK.md                   # Network configuration docs
├── PREREQS.md                   # Prerequisites documentation
└── README.md                    # Main documentation
```

#### Key Characteristics

**Separation of Concerns**:
- Clear separation between application code (`application/`) and infrastructure code (`infrastructure/`)
- Each Lambda function is self-contained with its own dependencies and Dockerfile
- Shared utilities are organized in dedicated subdirectories

**Documentation Strategy**:
- Multiple specialized documentation files for different aspects
- PREREQS.md for setup requirements
- NETWORK.md for network configuration
- CONTRIBUTING.md for team collaboration
- README.md as the main entry point

**Visual Documentation**:
- Dedicated `diagram/` directory with architecture visualizations
- Python scripts to generate diagrams programmatically
- Separate diagrams for each environment (dev, staging, production)

**Configuration Management**:
- Centralized `config.json` with environment-specific settings
- Supports Development, Staging, and Production environments
- Environment-specific tags, secrets, and ARNs

**Application Architecture**:
- Three distinct Lambda functions representing pipeline stages
- Each function has its own Dockerfile for containerized deployment
- Shared libraries and utilities organized in subdirectories
- SQL queries stored as separate files for maintainability

---

### 1.2 service-agreement-reconciliation-pre-processor

#### Project Overview
- **Purpose**: CDK deployment template (appears to be a template/starter project)
- **Technology Stack**: TypeScript, Node.js 22+, AWS CDK (TypeScript), Jest, ESLint
- **Deployment**: Multi-environment support (dev, staging, prod) via CDK stages
- **Architecture**: Modern TypeScript CDK project with workspace structure

#### Directory Structure

```
service-agreement-reconciliation-pre-processor/
├── .git/                          # Git version control
├── .github/                       # GitHub configuration
│   ├── workflows/                 # CI/CD pipeline definitions
│   ├── CODEOWNERS                 # Code ownership rules
│   └── dependabot.yml            # Dependency management
├── .husky/                        # Git hooks
│   └── pre-commit                # Pre-commit hook
├── .vscode/                       # VS Code configuration
│   └── extensions.json           # Recommended extensions
├── infrastructure/                # CDK infrastructure code
│   ├── @types/                   # TypeScript type definitions
│   │   └── config.d.ts          # Config type definitions
│   ├── bin/                      # CDK app entry point
│   │   └── app.ts               # Main app file
│   ├── lib/                      # Library code
│   │   ├── stacks/              # CDK stack definitions
│   │   ├── Environment.ts       # Environment configuration
│   │   └── utils.ts             # Utility functions
│   ├── test/                     # Infrastructure tests
│   │   └── bin/
│   ├── cdk.json                  # CDK configuration
│   └── package.json              # Infrastructure dependencies
├── .npmignore                     # NPM ignore rules
├── .npmrc                         # NPM configuration
├── catalog-info.yaml             # Backstage catalog metadata
├── eslint.config.mjs             # ESLint configuration
├── jest.config.json              # Jest test configuration
├── package-lock.json             # NPM lock file
├── package.json                  # Root package.json (workspace)
├── README.md                     # Main documentation
└── tsconfig.json                 # TypeScript configuration
```

#### Key Characteristics

**Modern TypeScript Setup**:
- Uses TypeScript throughout with strict type checking
- ESLint for code quality
- Jest for testing
- Husky for git hooks (pre-commit linting)

**Workspace Structure**:
- NPM workspaces with infrastructure as a workspace
- Centralized dependency management
- Monorepo-style organization

**Developer Experience**:
- VS Code extensions recommendations
- Pre-commit hooks for quality checks
- Lint-staged for efficient linting
- Watch mode support for development

**Infrastructure Organization**:
- All CDK code contained within `infrastructure/` directory
- Type definitions in `@types/` for better TypeScript support
- Separate `bin/` for app entry point and `lib/` for stack definitions
- Dedicated test directory for infrastructure tests

**Configuration Approach**:
- TypeScript-based configuration (config.ts)
- Type-safe configuration with TypeScript interfaces
- Environment-specific configuration via CDK stages

---

## Part 2: Comparative Analysis

### 2.1 Structural Comparison Matrix

| Component | risbi-data-pipeline | service-agreement | Notes |
|-----------|-------------------|------------------|-------|
| **Application Code Directory** | ✅ `application/` | ❌ Missing | risbi has separate Lambda code; service-agreement likely embeds in infrastructure |
| **Infrastructure Directory** | ✅ `infrastructure/` | ✅ `infrastructure/` | Both present but organized differently |
| **Diagram Directory** | ✅ `diagram/` | ❌ Missing | risbi has dedicated architecture diagrams |
| **Tests Directory** | ✅ `tests/` (root) | ✅ `infrastructure/test/` | Different locations; service-agreement tests are infrastructure-specific |
| **Documentation Files** | ✅ Multiple (PREREQS, NETWORK, CONTRIBUTING) | ⚠️ Partial (README only) | risbi has more comprehensive documentation |
| **GitHub Configuration** | ✅ `.github/` with workflows | ✅ `.github/` with workflows | Both have CI/CD |
| **Git Hooks** | ❌ Missing | ✅ `.husky/` | service-agreement has pre-commit hooks |
| **IDE Configuration** | ❌ Missing | ✅ `.vscode/` | service-agreement has VS Code setup |
| **Configuration Files** | ✅ `config.json` | ✅ TypeScript config | Different approaches (JSON vs TS) |
| **Dependency Management** | ✅ `requirements.txt` | ✅ `package.json` + workspaces | Language-specific differences |
| **Code Quality Tools** | ❌ Not visible | ✅ ESLint, Jest | service-agreement has explicit tooling |
| **Container Definitions** | ✅ Dockerfiles per Lambda | ❌ Not visible | risbi uses containerized Lambdas |

### 2.2 Key Differences

#### Architecture Pattern Differences

**risbi-data-pipeline**:
- Follows a **layered architecture** with clear separation between application and infrastructure
- Application code is completely separate from CDK infrastructure code
- Each Lambda function is a self-contained module with its own dependencies

**service-agreement**:
- Follows a **CDK-centric architecture** where infrastructure is the primary focus
- Application code (if any) is likely embedded within Lambda construct definitions
- More typical of a CDK template or infrastructure-focused project

#### Documentation Philosophy

**risbi-data-pipeline**:
- **Comprehensive multi-file approach**
- Specialized documentation for different concerns (prerequisites, network, contributing)
- Visual documentation with architecture diagrams
- Assumes complex setup and onboarding needs

**service-agreement**:
- **Single-file approach**
- All documentation in README.md
- Template-style documentation focused on getting started
- Assumes simpler setup or template usage

#### Development Workflow

**risbi-data-pipeline**:
- Python-based development workflow
- Manual quality checks (no visible pre-commit hooks)
- Focus on data pipeline development
- Separate testing of application vs infrastructure

**service-agreement**:
- TypeScript-based development workflow
- Automated quality checks via Husky pre-commit hooks
- Modern JavaScript tooling (ESLint, Jest, lint-staged)
- Integrated testing within infrastructure workspace

#### Configuration Strategy

**risbi-data-pipeline**:
- **JSON-based configuration** (`config.json`)
- Environment-specific sections (Development, Staging, Production)
- Centralized configuration with all environments in one file
- Runtime environment selection via CDK context

**service-agreement**:
- **TypeScript-based configuration** (`config.ts`)
- Type-safe configuration with interfaces
- Likely uses CDK stages for environment separation
- Compile-time type checking for configuration

---

## Part 3: Gap Analysis

### 3.1 Missing Components in service-agreement

#### High Priority Gaps

1. **Application Code Directory**
   - **Missing**: Dedicated `application/` directory
   - **Impact**: No clear separation between business logic and infrastructure
   - **Consequence**: Harder to maintain, test, and version application code independently

2. **Architecture Diagrams**
   - **Missing**: `diagram/` directory with visual documentation
   - **Impact**: No visual representation of system architecture
   - **Consequence**: Harder for new team members to understand system design

3. **Specialized Documentation**
   - **Missing**: PREREQS.md, NETWORK.md, CONTRIBUTING.md
   - **Impact**: All documentation crammed into single README
   - **Consequence**: Harder to find specific information, poor documentation organization

#### Medium Priority Gaps

4. **Root-Level Tests Directory**
   - **Missing**: Separate tests directory for application code
   - **Current**: Only infrastructure tests exist
   - **Impact**: No clear place for application-level tests if Lambda code is added

5. **Shared Utilities Organization**
   - **Missing**: Dedicated helpers or utilities directories
   - **Current**: Single `utils.ts` file
   - **Impact**: As project grows, utilities may become disorganized

6. **Environment-Specific Configuration**
   - **Missing**: Explicit multi-environment configuration file
   - **Current**: Configuration approach not clearly visible
   - **Impact**: May be harder to manage environment-specific settings

#### Low Priority Gaps

7. **Change Management Configuration**
   - **Missing**: `.changerc` file
   - **Impact**: No standardized change management process
   - **Note**: May not be needed depending on team workflow

### 3.2 Advantages of service-agreement Structure

While identifying gaps, it's important to note that service-agreement has some advantages:

1. **Modern Development Tooling**
   - Husky pre-commit hooks for quality gates
   - ESLint and Jest configured out of the box
   - VS Code integration for better developer experience

2. **Type Safety**
   - TypeScript throughout provides compile-time safety
   - Type definitions for configuration
   - Better IDE support and autocomplete

3. **Workspace Organization**
   - NPM workspaces for monorepo-style management
   - Cleaner dependency management
   - Better suited for multi-package projects

---

## Part 4: Alignment Recommendations

### 4.1 Recommended Additions (Prioritized)

#### Priority 1: Critical for Standardization

**1. Create Application Directory Structure**

```
service-agreement-reconciliation-pre-processor/
└── application/
    └── [function-name]/
        ├── src/
        │   └── index.ts          # Lambda handler
        ├── tests/
        │   └── index.test.ts     # Unit tests
        ├── Dockerfile            # If using container images
        ├── package.json          # Function-specific dependencies
        └── tsconfig.json         # Function-specific TS config
```

**Rationale**:
- Separates business logic from infrastructure code
- Makes Lambda functions independently testable and deployable
- Follows single responsibility principle
- Easier to version and maintain application code

**Implementation**:
- Create `application/` directory at root level
- Move any Lambda handler code from infrastructure to application
- Create subdirectories for each Lambda function
- Add function-specific package.json and dependencies

**2. Add Specialized Documentation Files**

Create the following files at root level:

**PREREQS.md**:
```markdown
## Prerequisites

### Required Access
- AWS account access with appropriate roles
- GitHub repository access
- [Other service access requirements]

### Local Development Requirements
- Node.js 22.13.0+
- AWS CLI configured
- AWS CDK CLI installed
- [Other tools]

### Setup Steps
1. Clone repository
2. Install dependencies
3. Configure AWS credentials
4. [Other setup steps]
```

**NETWORK.md**:
```markdown
## Network Configuration

### VPC Configuration
- VPC ID: [value]
- Subnets: [list]
- Security Groups: [list]

### External Connections
- [Service connections]
- [API endpoints]
- [Database connections]

### Firewall Rules
- [Ingress rules]
- [Egress rules]
```

**CONTRIBUTING.md**:
```markdown
## Contributing Guidelines

### Development Workflow
1. Create feature branch
2. Make changes
3. Run tests: `npm test`
4. Run linter: `npm run lint`
5. Create pull request

### Code Standards
- Follow TypeScript best practices
- Write unit tests for new features
- Update documentation

### Team Contacts
- [Team member] - [email]
- [Team member] - [email]
```

**Rationale**:
- Improves documentation discoverability
- Separates concerns (setup vs network vs contribution)
- Easier to maintain and update specific documentation
- Better onboarding experience for new team members

**3. Create Diagram Directory**

```
service-agreement-reconciliation-pre-processor/
└── diagram/
    ├── architecture.png          # Main architecture diagram
    ├── development.png           # Dev environment diagram
    ├── staging.png              # Staging environment diagram
    ├── production.png           # Production environment diagram
    └── README.md                # Diagram documentation
```

**Rationale**:
- Visual documentation is crucial for understanding complex systems
- Helps with architecture reviews and discussions
- Useful for onboarding and knowledge transfer
- Documents system design decisions

**Tools to Consider**:
- draw.io / diagrams.net (free, web-based)
- Lucidchart (commercial)
- Python diagrams library (code-based, like risbi-data-pipeline)
- AWS Architecture Icons (official AWS diagrams)

#### Priority 2: Important for Maintainability

**4. Organize Infrastructure Helpers**

```
service-agreement-reconciliation-pre-processor/
└── infrastructure/
    └── lib/
        ├── helpers/              # NEW
        │   ├── iam.ts           # IAM utilities
        │   ├── kms.ts           # KMS utilities
        │   ├── networking.ts    # VPC/networking utilities
        │   └── index.ts         # Barrel export
        ├── stacks/
        ├── Environment.ts
        └── utils.ts             # Keep for general utilities
```

**Rationale**:
- Better organization as project grows
- Easier to find specific utilities
- Follows risbi-data-pipeline pattern
- Improves code reusability

**5. Add Root-Level Tests Directory**

```
service-agreement-reconciliation-pre-processor/
└── tests/
    ├── integration/             # Integration tests
    ├── e2e/                     # End-to-end tests
    └── __init__.ts              # Test utilities
```

**Rationale**:
- Separates application tests from infrastructure tests
- Provides clear location for integration and E2E tests
- Matches risbi-data-pipeline structure
- Better test organization as project grows

**Note**: Keep `infrastructure/test/` for CDK infrastructure tests

#### Priority 3: Nice to Have

**6. Add Change Management Configuration**

Create `.changerc` if using conventional commits or changelog generation:

```json
{
  "changelog": true,
  "commit": true,
  "tag": true,
  "push": true,
  "publishConfig": {
    "access": "restricted"
  }
}
```

**Rationale**:
- Standardizes change management
- Automates changelog generation
- Improves release process
- Only needed if team uses conventional commits

### 4.2 Configuration Alignment

**Current State**:
- risbi-data-pipeline: Uses `config.json` with environment sections
- service-agreement: Uses TypeScript config files

**Recommendation**: Keep TypeScript configuration but enhance it

Create `infrastructure/lib/config.ts`:

```typescript
export interface EnvironmentConfig {
  environment: string;
  tags: Record<string, string>;
  alertEmail: string;
  // Add other environment-specific properties
}

export interface AppConfig {
  appName: string;
  tags: Record<string, string>;
}

export const config = {
  app: {
    appName: "service-agreement-reconciliation-pre-processor",
    tags: {
      "pfg-owner": "team-email@principal.com",
      "pfg-app-inventory-id": "appid-xxxxx",
      // ... other tags
    }
  },
  Development: {
    environment: "Development",
    tags: { "pfg-env-type": "Development" },
    alertEmail: "team-email@principal.com",
    // ... other dev-specific config
  },
  Staging: {
    environment: "Staging",
    tags: { "pfg-env-type": "Staging" },
    alertEmail: "team-email@principal.com",
    // ... other staging-specific config
  },
  Production: {
    environment: "Production",
    tags: { "pfg-env-type": "Production" },
    alertEmail: "team-email@principal.com",
    // ... other prod-specific config
  }
} as const;
```

**Rationale**:
- Maintains type safety of TypeScript
- Provides similar structure to risbi-data-pipeline
- Centralizes environment configuration
- Easier to manage multi-environment deployments

### 4.3 Structural Alignment Summary

**Final Recommended Structure**:

```
service-agreement-reconciliation-pre-processor/
├── .git/
├── .github/
│   ├── workflows/
│   ├── CODEOWNERS
│   └── dependabot.yml
├── .husky/                        # KEEP (advantage over risbi)
│   └── pre-commit
├── .vscode/                       # KEEP (advantage over risbi)
│   └── extensions.json
├── application/                   # ADD - Priority 1
│   └── [function-name]/
│       ├── src/
│       ├── tests/
│       ├── Dockerfile
│       └── package.json
├── diagram/                       # ADD - Priority 1
│   ├── architecture.png
│   ├── development.png
│   ├── staging.png
│   ├── production.png
│   └── README.md
├── infrastructure/
│   ├── @types/
│   ├── bin/
│   ├── lib/
│   │   ├── helpers/              # ADD - Priority 2
│   │   │   ├── iam.ts
│   │   │   ├── kms.ts
│   │   │   └── index.ts
│   │   ├── stacks/
│   │   ├── config.ts             # ENHANCE - Priority 2
│   │   ├── Environment.ts
│   │   └── utils.ts
│   ├── test/
│   ├── cdk.json
│   └── package.json
├── tests/                         # ADD - Priority 2
│   ├── integration/
│   └── e2e/
├── .changerc                      # ADD - Priority 3 (optional)
├── .npmignore
├── .npmrc
├── catalog-info.yaml
├── eslint.config.mjs
├── jest.config.json
├── package-lock.json
├── package.json
├── tsconfig.json
├── CONTRIBUTING.md                # ADD - Priority 1
├── NETWORK.md                     # ADD - Priority 1
├── PREREQS.md                     # ADD - Priority 1
└── README.md
```

---

## Part 5: Implementation Roadmap

### Phase 1: Documentation (Week 1)
1. Create PREREQS.md with setup requirements
2. Create NETWORK.md with network configuration
3. Create CONTRIBUTING.md with team guidelines
4. Update README.md to reference new documentation files

**Effort**: 4-8 hours
**Risk**: Low
**Dependencies**: None

### Phase 2: Visual Documentation (Week 1-2)
1. Create `diagram/` directory
2. Create architecture diagrams for each environment
3. Document diagram generation process
4. Add diagrams to README.md

**Effort**: 8-16 hours
**Risk**: Low
**Dependencies**: Architecture knowledge, diagramming tool selection

### Phase 3: Application Structure (Week 2-3)
1. Create `application/` directory structure
2. Move existing Lambda code (if any) to application directory
3. Set up function-specific package.json files
4. Update infrastructure code to reference new locations
5. Update build and deployment scripts

**Effort**: 16-24 hours
**Risk**: Medium (requires code refactoring)
**Dependencies**: Understanding of current Lambda implementations

### Phase 4: Infrastructure Organization (Week 3-4)
1. Create `infrastructure/lib/helpers/` directory
2. Move/create helper utilities (IAM, KMS, networking)
3. Enhance configuration with environment-specific settings
4. Create root-level `tests/` directory
5. Set up integration test framework

**Effort**: 12-20 hours
**Risk**: Medium (requires refactoring)
**Dependencies**: Phase 3 completion

### Phase 5: Optional Enhancements (Week 4+)
1. Add `.changerc` if using conventional commits
2. Set up automated changelog generation
3. Add additional tooling as needed

**Effort**: 4-8 hours
**Risk**: Low
**Dependencies**: Team workflow decisions

---

## Part 6: Considerations and Caveats

### 6.1 Language Differences

**Important**: Not all patterns from risbi-data-pipeline directly translate to TypeScript:

- **Python `requirements.txt`** → **TypeScript `package.json`**
- **Python modules** → **TypeScript modules with imports**
- **Python `__init__.py`** → **TypeScript `index.ts` (barrel exports)**
- **Python virtual environments** → **Node.js node_modules**

### 6.2 Project Purpose Differences

**risbi-data-pipeline**:
- Production data pipeline with complex ETL logic
- Multiple Lambda functions with distinct responsibilities
- Heavy focus on data processing and transformation

**service-agreement**:
- May have different architectural needs
- May not need multiple Lambda functions
- May have different complexity requirements

**Recommendation**: Adapt the structure to fit actual needs, not blindly copy

### 6.3 Team Workflow Considerations

Before implementing changes, consider:
- Current team size and expertise
- Existing development workflows
- CI/CD pipeline requirements
- Testing strategies
- Deployment frequency
- Maintenance burden

### 6.4 Incremental Adoption

**Don't try to implement everything at once**:
1. Start with documentation (low risk, high value)
2. Add diagrams (low risk, high value)
3. Refactor application structure (higher risk, plan carefully)
4. Enhance infrastructure organization (medium risk, medium value)

---

## Part 7: Conclusion

### Summary of Key Findings

1. **risbi-data-pipeline** has a more mature, production-ready structure with clear separation of concerns
2. **service-agreement** has modern development tooling but lacks organizational structure
3. Main gaps are: application directory, specialized documentation, and architecture diagrams
4. Both projects have strengths that can be learned from

### Recommended Next Steps

1. **Immediate** (This Week):
   - Create PREREQS.md, NETWORK.md, CONTRIBUTING.md
   - Start architecture diagram

2. **Short Term** (Next 2-4 Weeks):
   - Create application directory structure
   - Refactor Lambda code if needed
   - Organize infrastructure helpers

3. **Long Term** (Next 1-2 Months):
   - Establish testing strategy
   - Implement integration tests
   - Refine documentation based on team feedback

### Success Criteria

The alignment will be successful when:
- New team members can onboard using clear documentation
- Application code is separate from infrastructure code
- Architecture is visually documented
- Project structure is consistent with organizational standards
- Code is maintainable and testable

---

## Appendix: Quick Reference

### Directory Purpose Quick Reference

| Directory | Purpose | Priority |
|-----------|---------|----------|
| `application/` | Lambda function code and business logic | High |
| `diagram/` | Architecture diagrams and visual documentation | High |
| `infrastructure/` | CDK infrastructure code | Existing |
| `infrastructure/lib/helpers/` | Reusable infrastructure utilities | Medium |
| `tests/` | Application-level tests (integration, E2E) | Medium |
| `.github/` | CI/CD workflows and GitHub configuration | Existing |
| `.husky/` | Git hooks for quality gates | Existing |
| `.vscode/` | IDE configuration | Existing |

### Documentation File Quick Reference

| File | Purpose | Priority |
|------|---------|----------|
| README.md | Main project documentation and getting started | Existing |
| PREREQS.md | Prerequisites and setup requirements | High |
| NETWORK.md | Network configuration and connectivity | High |
| CONTRIBUTING.md | Contribution guidelines and team contacts | High |
| diagram/README.md | Diagram documentation and generation | Medium |

---

**Document Version**: 1.0
**Last Updated**: February 9, 2026
**Author**: Kiro Analysis
