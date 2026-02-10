# Design Document: Service Agreement Project Restructuring

## Overview

This design document outlines the approach for restructuring the service-agreement-reconciliation-pre-processor TypeScript CDK project to align with the organizational structure of the risbi-data-pipeline-from-snowflake-to-db2 Python CDK project. The restructuring will improve maintainability, documentation clarity, and separation of concerns while preserving all existing functionality and TypeScript tooling advantages.

### Goals

1. **Structural Alignment**: Reorganize directories to match the risbi-pipeline pattern
2. **Documentation Enhancement**: Create comprehensive, specialized documentation files
3. **Separation of Concerns**: Separate application code from infrastructure code
4. **Scalability**: Enable easy addition of new Lambda functions and infrastructure
5. **Local Development**: Ensure the project can be instantiated and validated locally
6. **Functionality Preservation**: Maintain all existing CDK functionality

### Non-Goals

- Rewriting existing functionality
- Changing the programming language (remains TypeScript)
- Modifying AWS resource configurations
- Changing deployment pipelines or CI/CD workflows

### Constraints

- Must maintain TypeScript and Node.js tooling (ESLint, Jest, Husky)
- Must preserve existing VS Code configuration
- Must remain compatible with existing AWS CDK version
- Must be verifiable through `cdk synth` and `cdk deploy`
- Changes must be incremental and testable
- Must support local development without private registry authentication
- Must distinguish between local development and production deployment registry requirements

---

## Architecture

### Current Architecture

The service-agreement project currently follows a CDK-centric architecture:

```
service-agreement-reconciliation-pre-processor/
├── infrastructure/          # All CDK code
│   ├── bin/app.ts          # CDK app entry point
│   ├── lib/                # Stacks and utilities
│   └── test/               # Infrastructure tests
├── .husky/                 # Git hooks
├── .vscode/                # IDE config
└── [root config files]     # TypeScript, Jest, ESLint configs
```

**Characteristics**:
- Infrastructure-focused with minimal application code
- Single README.md for all documentation
- Utilities in a single utils.ts file
- Tests only for infrastructure

### Target Architecture

The restructured project will follow a layered architecture with clear separation:

```
service-agreement-reconciliation-pre-processor/
├── application/            # Lambda functions and business logic
│   └── [function-name]/
│       ├── src/           # Function source code
│       ├── tests/         # Function unit tests
│       ├── package.json   # Function dependencies
│       └── tsconfig.json  # Function TS config
├── diagram/               # Architecture diagrams
│   ├── architecture.png
│   └── README.md
├── infrastructure/        # CDK infrastructure code
│   ├── bin/app.ts        # CDK app entry point
│   ├── lib/
│   │   ├── helpers/      # Reusable utilities
│   │   │   ├── iam.ts
│   │   │   ├── kms.ts
│   │   │   ├── networking.ts
│   │   │   └── index.ts
│   │   ├── stacks/       # CDK stacks
│   │   ├── config.ts     # Enhanced configuration
│   │   └── Environment.ts
│   └── test/             # Infrastructure tests
├── tests/                # Integration and E2E tests
│   ├── integration/
│   └── e2e/
├── PREREQS.md            # Prerequisites documentation
├── NETWORK.md            # Network configuration
├── CONTRIBUTING.md       # Contribution guidelines
└── README.md             # Main documentation
```

**Characteristics**:
- Clear separation between application and infrastructure layers
- Specialized documentation for different concerns
- Organized helper utilities for reusability
- Scalable structure for adding new functions
- Multiple test directories for different test types

### Architectural Principles

1. **Separation of Concerns**: Application logic is separate from infrastructure definitions
2. **Single Responsibility**: Each directory has a clear, focused purpose
3. **Scalability**: Structure supports growth without reorganization
4. **Discoverability**: Clear naming and organization makes code easy to find
5. **Type Safety**: Maintain TypeScript's type safety throughout

---

## Components and Interfaces

### 1. Application Layer

**Purpose**: Contains Lambda function code and business logic, separate from infrastructure.

**Structure**:
```
application/
└── [function-name]/
    ├── src/
    │   ├── index.ts           # Lambda handler entry point
    │   ├── [business-logic].ts # Business logic modules
    │   └── types.ts           # Type definitions
    ├── tests/
    │   ├── index.test.ts      # Handler tests
    │   └── [module].test.ts   # Module tests
    ├── package.json           # Function-specific dependencies
    ├── tsconfig.json          # Function-specific TypeScript config
    └── README.md              # Function documentation
```

**Key Interfaces**:

```typescript
// Lambda handler interface
export interface LambdaHandler {
  (event: any, context: Context): Promise<any>;
}

// Function configuration
export interface FunctionConfig {
  name: string;
  handler: string;
  runtime: Runtime;
  timeout: Duration;
  memorySize: number;
  environment?: { [key: string]: string };
}
```

**Design Decisions**:
- Each function is self-contained with its own dependencies
- Functions can be developed and tested independently
- TypeScript configuration can be customized per function
- Clear entry point (index.ts) for Lambda handlers
- Function package.json uses public npm registry for local development

### 2. NPM Registry Configuration

**Purpose**: Enables local development without private registry authentication while supporting production deployments with private registries.

**Configuration Files**:

**.npmrc-local** (for local development):
```ini
# Local development configuration - uses public npm registry
registry=https://registry.npmjs.org/
# No authentication required for local development
```

**.npmrc-prod** (for production deployment):
```ini
# Production configuration - uses private Artifactory registry
registry=https://packages.principal.com/artifactory/api/npm/pfg-npm-dev-virt/
//packages.principal.com/:_auth=${PFG_ARTIFACTORY_NPM_AUTH}
ignore-scripts=true
```

**.gitignore** (updated to include):
```
# NPM configuration with credentials
.npmrc

# Node modules
node_modules/
```

**Setup Instructions in PREREQS.md**:
```markdown
## NPM Registry Configuration

### For Local Development

Copy the local development configuration:
```bash
cp .npmrc-local .npmrc
```

This uses the public npm registry and requires no authentication.

### For Production Deployment

Copy the production configuration:
```bash
cp .npmrc-prod .npmrc
```

Set the authentication token:
```bash
export PFG_ARTIFACTORY_NPM_AUTH="your-auth-token"
```

**Important**: Never commit `.npmrc` with credentials to version control!
```

**Design Decisions**:
- Separate configuration files for different environments
- .npmrc is gitignored to prevent credential leaks
- Template files (.npmrc-local, .npmrc-prod) are committed
- Local development defaults to public registry
- Production can use private registry when needed
- Clear documentation in PREREQS.md for setup

### 3. Documentation Layer

**Purpose**: Provides comprehensive, specialized documentation for different aspects of the project.

**Components**:

**PREREQS.md**:
```markdown
## Prerequisites

### Required Access
- AWS account access with appropriate IAM roles
- GitHub repository access
- [Service-specific access]

### Local Development Requirements
- Node.js 22.13.0+
- AWS CLI v2+ configured with credentials
- AWS CDK CLI (npm install -g aws-cdk)
- Git

### NPM Registry Configuration

#### For Local Development (Default)
Copy the local development configuration:
```bash
cp .npmrc-local .npmrc
```

This uses the public npm registry (registry.npmjs.org) and requires no authentication.

#### For Production Deployment
Copy the production configuration:
```bash
cp .npmrc-prod .npmrc
```

Set the authentication token:
```bash
export PFG_ARTIFACTORY_NPM_AUTH="your-auth-token"
```

**Important**: Never commit `.npmrc` with credentials to version control!

### Setup Steps
1. Clone repository: git clone [repo-url]
2. Configure NPM registry: cp .npmrc-local .npmrc
3. Install dependencies: npm install
4. Configure AWS credentials: aws configure
5. Bootstrap CDK (first time): cdk bootstrap
6. Synthesize: npm run cdk synth
```

**NETWORK.md**:
```markdown
## Network Configuration

### VPC Configuration
- VPC ID: [to be configured]
- Subnets: [to be configured]
- Security Groups: [to be configured]

### Lambda Network Configuration
- VPC Enabled: [Yes/No]
- Subnet Type: [Private/Public]
- Security Group Rules: [to be documented]

### External Connections
- AWS Services: [S3, DynamoDB, etc.]
- External APIs: [if any]
- Database Connections: [if any]
```

**CONTRIBUTING.md**:
```markdown
## Contributing Guidelines

### Development Workflow
1. Create feature branch: git checkout -b feature/your-feature
2. Configure NPM for local dev: cp .npmrc-local .npmrc
3. Make changes
4. Run linter: npm run lint
5. Run tests: npm test
6. Commit (triggers pre-commit hooks)
7. Push and create PR

### NPM Registry Configuration
- **Local Development**: Use `.npmrc-local` (public registry, no auth)
- **Production Deployment**: Use `.npmrc-prod` (private registry, requires auth)
- **Never commit** `.npmrc` with credentials
- The `.npmrc` file is gitignored for security

### Code Standards
- Follow TypeScript best practices
- Use ESLint configuration
- Write unit tests for new features
- Update documentation

### Adding New Lambda Functions
1. Create directory: application/[function-name]
2. Copy template structure
3. Implement handler in src/index.ts
4. Add tests in tests/
5. Update infrastructure to reference function

### Team Contacts
- Team Lead: [name] - [email]
- DevOps: [name] - [email]
```

**diagram/README.md**:
```markdown
## Architecture Diagrams

### Diagrams
- architecture.png: Overall system architecture
- [environment].png: Environment-specific diagrams

### Updating Diagrams
1. Use draw.io or preferred tool
2. Export as PNG
3. Place in diagram/ directory
4. Update this README with description

### Diagram Standards
- Use AWS Architecture Icons
- Include all AWS services
- Show data flow with arrows
- Label all connections
```

### 3. Infrastructure Layer - Helpers

**Purpose**: Provides reusable utility functions for common AWS service patterns.

**Structure**:
```
infrastructure/lib/helpers/
├── iam.ts           # IAM role and policy utilities
├── kms.ts           # KMS key utilities
├── networking.ts    # VPC and networking utilities
└── index.ts         # Barrel exports
```

**Key Interfaces**:

```typescript
// helpers/iam.ts
export interface RoleConfig {
  roleName: string;
  assumedBy: IPrincipal;
  managedPolicies?: IManagedPolicy[];
  inlinePolicies?: { [name: string]: PolicyDocument };
}

export function createLambdaRole(
  scope: Construct,
  id: string,
  config: RoleConfig
): Role;

// helpers/kms.ts
export interface KmsKeyConfig {
  alias: string;
  description: string;
  enableKeyRotation?: boolean;
  removalPolicy?: RemovalPolicy;
}

export function createKmsKey(
  scope: Construct,
  id: string,
  config: KmsKeyConfig
): Key;

// helpers/networking.ts
export interface VpcConfig {
  vpcId?: string;
  subnetIds?: string[];
  securityGroupIds?: string[];
}

export function getVpcFromConfig(
  scope: Construct,
  config: VpcConfig
): IVpc;

// helpers/index.ts
export * from './iam';
export * from './kms';
export * from './networking';
```

**Design Decisions**:
- Helpers are pure functions or factory functions
- Each helper module focuses on a single AWS service
- Barrel export (index.ts) for convenient imports
- Type-safe interfaces for all configurations

### 4. Infrastructure Layer - Configuration

**Purpose**: Provides centralized, type-safe configuration for multi-environment deployments.

**Enhanced Configuration Structure**:

```typescript
// infrastructure/lib/config.ts

export interface EnvironmentConfig {
  region: string;
  account: string;
  tags: Tags;
  alertEmail?: string;
  vpcConfig?: VpcConfig;
  // Environment-specific settings
}

export interface Tags {
  [key: string]: string;
}

export interface AppConfig {
  appName: string;
  tags: Tags;
}

export interface Configuration {
  app: AppConfig;
  Development: EnvironmentConfig;
  Staging: EnvironmentConfig;
  Production: EnvironmentConfig;
  // DR environments
  DevelopmentDr?: EnvironmentConfig;
  StagingDr?: EnvironmentConfig;
  ProductionDr?: EnvironmentConfig;
}

export const config: Configuration = {
  app: {
    appName: "service-agreement-reconciliation-pre-processor",
    tags: {
      "pfg-service-id": "idk",
      "pfg-data-class": "internal",
      "pfg-privacy-class": "not-personal",
      "pfg-recovery-method": "restore",
      "pfg-data-retention": "p20",
    }
  },
  Development: {
    region: "us-east-1",
    account: "815931739794",
    tags: {
      "pfg-env-type": "Development"
    },
    alertEmail: "team-dev@principal.com"
  },
  Staging: {
    region: "us-east-1",
    account: "545750752196",
    tags: {
      "pfg-env-type": "Staging"
    },
    alertEmail: "team-staging@principal.com"
  },
  Production: {
    region: "us-east-1",
    account: "720377205895",
    tags: {
      "pfg-env-type": "Production"
    },
    alertEmail: "team-prod@principal.com"
  },
  DevelopmentDr: {
    region: "us-west-2",
    account: "815931739794",
    tags: {
      "pfg-env-type": "Development"
    }
  },
  StagingDr: {
    region: "us-west-2",
    account: "545750752196",
    tags: {
      "pfg-env-type": "Staging"
    }
  },
  ProductionDr: {
    region: "us-west-2",
    account: "720377205895",
    tags: {
      "pfg-env-type": "Production"
    }
  }
};

// Helper function to get environment config
export function getEnvironmentConfig(env: string): EnvironmentConfig {
  const envConfig = config[env as keyof typeof config];
  if (!envConfig || typeof envConfig === 'object' && 'appName' in envConfig) {
    throw new Error(`Invalid environment: ${env}`);
  }
  return envConfig as EnvironmentConfig;
}

// Helper function to merge tags
export function getEnvironmentTags(env: string): Tags {
  const envConfig = getEnvironmentConfig(env);
  return {
    ...config.app.tags,
    ...envConfig.tags
  };
}
```

**Design Decisions**:
- Centralized configuration similar to risbi-pipeline's config.json
- Type-safe with TypeScript interfaces
- Helper functions for common operations
- Supports multiple environments including DR
- Extensible for adding new environment-specific properties

### 5. Testing Layer

**Purpose**: Provides organized structure for different types of tests.

**Structure**:
```
tests/
├── integration/
│   ├── lambda-integration.test.ts
│   └── stack-integration.test.ts
├── e2e/
│   ├── deployment.test.ts
│   └── workflow.test.ts
└── helpers/
    └── test-utils.ts

infrastructure/test/
├── bin/
│   └── app.test.ts
└── stacks/
    └── infrastructure.test.ts

application/[function-name]/tests/
└── index.test.ts
```

**Test Types**:

1. **Unit Tests** (in application/*/tests/ and infrastructure/test/):
   - Test individual functions and components
   - Fast execution
   - No external dependencies

2. **Integration Tests** (in tests/integration/):
   - Test interaction between components
   - May use AWS SDK
   - Test CDK synthesis output

3. **End-to-End Tests** (in tests/e2e/):
   - Test complete workflows
   - May require deployed resources
   - Validate actual AWS behavior

**Test Utilities**:

```typescript
// tests/helpers/test-utils.ts

export interface MockEvent {
  [key: string]: any;
}

export function createMockLambdaEvent(overrides?: Partial<MockEvent>): MockEvent {
  return {
    // Default mock event structure
    ...overrides
  };
}

export function createMockContext(): Context {
  return {
    // Mock context
  } as Context;
}
```

---

## Data Models

### Configuration Data Model

The configuration system uses TypeScript interfaces to define the structure of environment-specific settings:

```typescript
interface Configuration {
  app: AppConfig;
  [environment: string]: EnvironmentConfig | AppConfig;
}

interface AppConfig {
  appName: string;
  tags: Tags;
}

interface EnvironmentConfig {
  region: string;
  account: string;
  tags: Tags;
  alertEmail?: string;
  vpcConfig?: VpcConfig;
}

interface Tags {
  [key: string]: string;
}
```

**Relationships**:
- Configuration contains one AppConfig and multiple EnvironmentConfigs
- Each EnvironmentConfig inherits app-level tags and can override/extend them
- Tags are merged: app tags + environment tags

### Lambda Function Data Model

Each Lambda function follows a consistent structure:

```typescript
interface LambdaFunction {
  name: string;
  path: string;              // Path to application/[function-name]
  handler: string;           // e.g., "index.handler"
  runtime: Runtime;
  config: FunctionConfig;
}

interface FunctionConfig {
  timeout: Duration;
  memorySize: number;
  environment?: EnvironmentVariables;
  vpc?: VpcConfig;
  role?: RoleConfig;
}

interface EnvironmentVariables {
  [key: string]: string;
}
```

### Directory Structure Data Model

The project structure can be represented as:

```typescript
interface ProjectStructure {
  application: ApplicationLayer;
  diagram: DiagramLayer;
  infrastructure: InfrastructureLayer;
  tests: TestLayer;
  documentation: DocumentationLayer;
}

interface ApplicationLayer {
  functions: LambdaFunction[];
}

interface DiagramLayer {
  diagrams: Diagram[];
}

interface InfrastructureLayer {
  bin: string;              // Entry point
  lib: {
    helpers: Helper[];
    stacks: Stack[];
    config: Configuration;
  };
  test: Test[];
}

interface TestLayer {
  integration: Test[];
  e2e: Test[];
}

interface DocumentationLayer {
  prereqs: string;
  network: string;
  contributing: string;
  readme: string;
}
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After analyzing all acceptance criteria, I identified the following testable properties. Many criteria are examples (specific file/directory checks) rather than universal properties. The properties below focus on rules that should hold across multiple instances (e.g., all Lambda functions, all helper modules, all environments).

**Redundancy Analysis**:
- Properties 3.3 and 3.4 (Lambda functions having package.json and tests) could be combined into a single "Lambda function structure" property
- Properties 5.5 and 5.6 (test location) are complementary and should remain separate
- Property 10.6 (CDK synth equivalence) subsumes checking individual resource configurations
- Configuration properties (6.6) apply to all environments and should be a single property

### Properties

**Property 1: Lambda Function Structure Consistency**

*For any* Lambda function directory in the application layer, the directory SHALL contain a package.json file with function-specific dependencies AND a tests subdirectory for unit tests.

**Validates: Requirements 3.3, 3.4**

---

**Property 2: Containerized Lambda Dockerfile Presence**

*For any* Lambda function that uses container images, the function directory SHALL include a Dockerfile.

**Validates: Requirements 3.5**

---

**Property 3: Infrastructure References Application Code**

*For any* Lambda function defined in the infrastructure layer, the infrastructure code SHALL correctly reference the Lambda code from the application directory, and TypeScript compilation SHALL succeed.

**Validates: Requirements 3.6, 10.3**

---

**Property 4: Helper Module Independence**

*For any* helper utility module in the infrastructure/lib/helpers directory, the module SHALL be importable independently without requiring other helper modules, ensuring that adding new helpers does not affect existing ones.

**Validates: Requirements 4.7, 9.3**

---

**Property 5: Application Test Location**

*For any* application code that exists, the corresponding unit tests SHALL reside in the application directory structure (specifically in application/[function-name]/tests/), not in the root tests directory.

**Validates: Requirements 5.5**

---

**Property 6: Integration Test Location**

*For any* integration test that exists, the test SHALL reside in the root tests/integration directory, not in the infrastructure test directory or application test directories.

**Validates: Requirements 5.6**

---

**Property 7: Environment Configuration Completeness**

*For any* environment defined in the configuration (Development, Staging, Production, and their DR counterparts), the environment SHALL include environment-specific tags that identify the environment type.

**Validates: Requirements 6.6**

---

**Property 8: Lambda Function Independence**

*For any* Lambda function in the application directory, the function SHALL be self-contained such that adding, modifying, or removing the function does not require changes to other Lambda function code.

**Validates: Requirements 9.1**

---

**Property 9: Configuration Extensibility**

*For any* new environment-specific property added to the configuration interfaces, the TypeScript compiler SHALL successfully compile the configuration, ensuring type safety is maintained.

**Validates: Requirements 9.5**

---

**Property 10: CDK Stack Functionality Preservation**

*For any* CDK stack that existed before restructuring, the stack SHALL maintain identical functionality after restructuring, verified by comparing the synthesized CloudFormation templates for functional equivalence.

**Validates: Requirements 10.1, 10.6**

---

**Property 11: Lambda Function Functionality Preservation**

*For any* Lambda function that existed before restructuring, the function SHALL maintain identical behavior after restructuring, verified by running the function's unit tests successfully.

**Validates: Requirements 10.2, 10.5**

---

### Example-Based Tests

The following acceptance criteria are best validated through specific examples rather than universal properties:

**Documentation Structure Examples**:
- Verify PREREQS.md exists with required sections (Requirements 1.1, 1.5)
- Verify NETWORK.md exists with required sections (Requirements 1.2, 1.6)
- Verify CONTRIBUTING.md exists with required sections (Requirements 1.3, 1.7)
- Verify README.md references other documentation (Requirements 1.4)

**Directory Structure Examples**:
- Verify diagram/ directory exists at root (Requirements 2.1)
- Verify application/ directory exists at root (Requirements 3.1)
- Verify infrastructure/lib/helpers/ directory exists (Requirements 4.1)
- Verify tests/ directory exists at root with integration/ and e2e/ subdirectories (Requirements 5.1, 5.2, 5.3)
- Verify infrastructure/test/ directory still exists (Requirements 5.4)

**Configuration Examples**:
- Verify config.ts exists with TypeScript interfaces (Requirements 6.1, 6.2)
- Verify config includes Development, Staging, Production sections (Requirements 6.3)
- Verify config includes app-level properties (Requirements 6.5)
- Verify TypeScript compilation succeeds (Requirements 6.7)

**Local Development Examples**:
- Verify npm install completes without errors (Requirements 8.1, 8.5)
- Verify cdk synth succeeds (Requirements 8.2, 11.3)
- Verify build completes successfully (Requirements 8.6)
- Verify npm test runs and reports results (Requirements 8.7, 10.4)

**Tooling Preservation Examples**:
- Verify package.json exists (not requirements.txt) (Requirements 12.1)
- Verify index.ts used for barrel exports (Requirements 12.2)
- Verify ESLint and Jest configs exist (Requirements 12.3)
- Verify .husky directory exists (Requirements 12.4)
- Verify .vscode directory exists (Requirements 12.5)

---

## Error Handling

### File System Operations

**Error Scenarios**:
1. **Directory Creation Failures**: If directories cannot be created due to permissions
2. **File Move Failures**: If files cannot be moved due to locks or permissions
3. **File Write Failures**: If documentation files cannot be written

**Handling Strategy**:
- Validate permissions before starting restructuring
- Use atomic operations where possible
- Provide clear error messages indicating which operation failed
- Maintain ability to rollback changes if errors occur
- Log all file system operations for debugging

### CDK Synthesis Failures

**Error Scenarios**:
1. **Invalid TypeScript**: Code doesn't compile after restructuring
2. **Missing Dependencies**: Imports cannot be resolved
3. **Invalid CDK Constructs**: CDK synthesis fails due to invalid configurations

**Handling Strategy**:
- Run TypeScript compilation after each major change
- Run `cdk synth` incrementally to catch issues early
- Validate all import paths after moving files
- Keep backup of original structure until verification complete

### Configuration Errors

**Error Scenarios**:
1. **Type Mismatches**: Configuration doesn't match TypeScript interfaces
2. **Missing Required Fields**: Environment configs missing required properties
3. **Invalid Environment Names**: References to non-existent environments

**Handling Strategy**:
- Leverage TypeScript's type checking for compile-time validation
- Provide default values for optional configuration
- Validate environment names against known environments
- Clear error messages for configuration issues

### Test Failures

**Error Scenarios**:
1. **Broken Test Imports**: Tests can't import moved code
2. **Changed Test Paths**: Test discovery fails after restructuring
3. **Functionality Regressions**: Tests fail due to broken functionality

**Handling Strategy**:
- Update test imports immediately after moving code
- Update Jest configuration for new test locations
- Run tests after each major change
- Don't proceed if tests fail

---

## Testing Strategy

### Dual Testing Approach

This restructuring project requires both **unit tests** and **property-based tests** to ensure correctness:

- **Unit tests**: Verify specific examples, directory structures, and file existence
- **Property tests**: Verify universal rules that apply across multiple instances

Both testing approaches are complementary and necessary for comprehensive validation.

### Unit Testing

**Focus Areas**:
1. **Directory Structure Validation**
   - Verify specific directories exist (application/, diagram/, tests/, etc.)
   - Verify specific files exist (PREREQS.md, NETWORK.md, CONTRIBUTING.md, etc.)
   - Verify file content contains required sections

2. **Configuration Validation**
   - Verify config.ts exists and compiles
   - Verify specific environments are defined
   - Verify required configuration properties exist

3. **Build and Deployment Validation**
   - Verify `npm install` succeeds
   - Verify `npm run build` succeeds
   - Verify `cdk synth` succeeds
   - Verify `npm test` runs successfully

4. **Tooling Preservation**
   - Verify ESLint configuration exists
   - Verify Jest configuration exists
   - Verify Husky hooks exist
   - Verify VS Code configuration exists

**Test Organization**:
- Place restructuring validation tests in `tests/integration/restructuring-validation.test.ts`
- Use Jest as the test framework
- Run tests after each major restructuring step

### Property-Based Testing

**Property Testing Library**: Use `fast-check` for TypeScript property-based testing

**Configuration**: Each property test should run a minimum of 100 iterations

**Test Tagging**: Each property test must include a comment referencing the design property:
```typescript
// Feature: service-agreement-restructure, Property 1: Lambda Function Structure Consistency
```

**Focus Areas**:

1. **Lambda Function Structure (Property 1)**
   - Generate random Lambda function names
   - Verify each has package.json and tests/ directory
   - Tag: `Feature: service-agreement-restructure, Property 1: Lambda Function Structure Consistency`

2. **Infrastructure References (Property 3)**
   - For all Lambda functions in infrastructure code
   - Verify imports resolve correctly
   - Verify TypeScript compilation succeeds
   - Tag: `Feature: service-agreement-restructure, Property 3: Infrastructure References Application Code`

3. **Helper Module Independence (Property 4)**
   - For all helper modules
   - Verify each can be imported independently
   - Verify no circular dependencies
   - Tag: `Feature: service-agreement-restructure, Property 4: Helper Module Independence`

4. **Test Location Properties (Properties 5, 6)**
   - For all application tests, verify location in application directory
   - For all integration tests, verify location in tests/integration
   - Tag: `Feature: service-agreement-restructure, Property 5: Application Test Location`
   - Tag: `Feature: service-agreement-restructure, Property 6: Integration Test Location`

5. **Environment Configuration (Property 7)**
   - For all defined environments
   - Verify each has required tags
   - Tag: `Feature: service-agreement-restructure, Property 7: Environment Configuration Completeness`

6. **Lambda Independence (Property 8)**
   - For all Lambda functions
   - Verify no cross-function dependencies
   - Tag: `Feature: service-agreement-restructure, Property 8: Lambda Function Independence`

7. **Functionality Preservation (Properties 10, 11)**
   - Compare CDK synth output before and after
   - Run all Lambda function tests
   - Tag: `Feature: service-agreement-restructure, Property 10: CDK Stack Functionality Preservation`
   - Tag: `Feature: service-agreement-restructure, Property 11: Lambda Function Functionality Preservation`

### Test Execution Strategy

**Incremental Testing**:
1. Run unit tests after creating each directory
2. Run TypeScript compilation after moving each file
3. Run `cdk synth` after major structural changes
4. Run property tests after completing each major phase
5. Run full test suite before final verification

**Verification Checkpoints**:
- After documentation creation: Verify files exist and have content
- After directory creation: Verify structure matches target
- After code movement: Verify compilation and imports
- After configuration update: Verify type safety
- Final verification: Run all tests and `cdk synth`

### Test Implementation Notes

**Property Test Example**:
```typescript
import fc from 'fast-check';

// Feature: service-agreement-restructure, Property 1: Lambda Function Structure Consistency
describe('Lambda Function Structure', () => {
  it('should have package.json and tests directory for all functions', () => {
    fc.assert(
      fc.property(
        fc.array(fc.string(), { minLength: 1, maxLength: 10 }),
        (functionNames) => {
          functionNames.forEach(name => {
            const functionPath = `application/${name}`;
            expect(fs.existsSync(`${functionPath}/package.json`)).toBe(true);
            expect(fs.existsSync(`${functionPath}/tests`)).toBe(true);
          });
        }
      ),
      { numRuns: 100 }
    );
  });
});
```

**Unit Test Example**:
```typescript
describe('Directory Structure', () => {
  it('should have application directory at root', () => {
    expect(fs.existsSync('application')).toBe(true);
  });

  it('should have diagram directory at root', () => {
    expect(fs.existsSync('diagram')).toBe(true);
  });

  it('should have tests directory at root', () => {
    expect(fs.existsSync('tests')).toBe(true);
  });
});
```

---

## Implementation Phases

### Phase 1: Documentation (Low Risk)

**Objective**: Create specialized documentation files

**Steps**:
1. Create PREREQS.md with prerequisites and setup instructions
2. Create NETWORK.md with network configuration
3. Create CONTRIBUTING.md with contribution guidelines
4. Update README.md to reference new documentation
5. Create diagram/README.md

**Verification**: Verify files exist and contain required sections

**Risk**: Low - No code changes

---

### Phase 2: Directory Structure (Low Risk)

**Objective**: Create new directory structure

**Steps**:
1. Create application/ directory at root
2. Create diagram/ directory at root
3. Create tests/ directory at root with integration/ and e2e/ subdirectories
4. Create infrastructure/lib/helpers/ directory
5. Create infrastructure/lib/helpers/index.ts for barrel exports

**Verification**: Verify all directories exist, run `cdk synth` to ensure no breakage

**Risk**: Low - Only adding directories, not moving code

---

### Phase 3: Configuration Enhancement (Medium Risk)

**Objective**: Create enhanced configuration file

**Steps**:
1. Create infrastructure/lib/config.ts with TypeScript interfaces
2. Migrate configuration from bin/app.ts to config.ts
3. Update bin/app.ts to import from config.ts
4. Add helper functions for environment config and tags
5. Update type definitions in @types/config.d.ts if needed

**Verification**: Run TypeScript compilation, run `cdk synth`

**Risk**: Medium - Modifying configuration structure

---

### Phase 4: Helper Utilities (Medium Risk)

**Objective**: Create helper utility modules

**Steps**:
1. Create infrastructure/lib/helpers/iam.ts (if IAM utilities needed)
2. Create infrastructure/lib/helpers/kms.ts (if KMS utilities needed)
3. Create infrastructure/lib/helpers/networking.ts (if networking utilities needed)
4. Update infrastructure/lib/helpers/index.ts to export all helpers
5. Update infrastructure code to use helpers if applicable

**Verification**: Run TypeScript compilation, run `cdk synth`

**Risk**: Medium - Adding new code that may be used by infrastructure

---

### Phase 5: Application Layer (High Risk - if Lambda code exists)

**Objective**: Move Lambda code to application directory

**Steps**:
1. Identify existing Lambda function code (if any)
2. Create application/[function-name]/ directories
3. Move Lambda code to application/[function-name]/src/
4. Create application/[function-name]/package.json
5. Create application/[function-name]/tests/ directory
6. Move Lambda tests to application/[function-name]/tests/
7. Update infrastructure code to reference new Lambda locations
8. Update import paths

**Verification**: Run TypeScript compilation, run tests, run `cdk synth`

**Risk**: High if Lambda code exists - Requires careful path updates

**Note**: If no Lambda code currently exists, this phase creates the structure for future Lambda functions

---

### Phase 6: Testing Structure (Low Risk)

**Objective**: Organize tests into appropriate directories

**Steps**:
1. Identify integration tests (if any)
2. Move integration tests to tests/integration/
3. Create test utilities in tests/helpers/ if needed
4. Update Jest configuration for new test locations
5. Ensure infrastructure tests remain in infrastructure/test/

**Verification**: Run `npm test`, verify all tests are discovered and pass

**Risk**: Low - Moving tests, Jest should discover them

---

### Phase 7: Final Verification (Critical)

**Objective**: Verify all functionality is preserved

**Steps**:
1. Run full test suite: `npm test`
2. Run TypeScript compilation: `npm run build`
3. Run CDK synthesis: `cdk synth`
4. Compare synthesized CloudFormation templates with pre-restructuring templates
5. Verify all directories and files are in place
6. Run property-based tests
7. Document any differences or issues

**Verification**: All tests pass, `cdk synth` succeeds, templates are functionally equivalent

**Risk**: Critical - Final validation of entire restructuring

---

## Rollback Strategy

If any phase fails verification:

1. **Stop immediately** - Do not proceed to next phase
2. **Identify the issue** - Determine what caused the failure
3. **Fix or rollback** - Either fix the issue or rollback the phase
4. **Re-verify** - Run verification steps again
5. **Document** - Document the issue and resolution

**Rollback Approach**:
- Use Git to track changes at each phase
- Commit after each successful phase
- Can rollback to previous commit if needed
- Keep backup of original structure until final verification

---

## Success Criteria

The restructuring is successful when:

1. ✅ All documentation files exist with required content
2. ✅ All directories match the target structure
3. ✅ Configuration is centralized and type-safe
4. ✅ Helper utilities are organized and reusable
5. ✅ Application code (if any) is separated from infrastructure
6. ✅ Tests are organized by type
7. ✅ `npm install` completes without errors
8. ✅ `npm run build` completes without errors
9. ✅ `npm test` runs and all tests pass
10. ✅ `cdk synth` succeeds
11. ✅ Synthesized CloudFormation templates are functionally equivalent to pre-restructuring
12. ✅ All property-based tests pass
13. ✅ Project can be deployed with `cdk deploy`
14. ✅ TypeScript tooling (ESLint, Jest, Husky) still works
15. ✅ VS Code configuration is preserved

---

## Appendix: File Templates

### PREREQS.md Template

```markdown
# Prerequisites

## Required Access

- AWS account access with appropriate IAM roles
  - Development account: 815931739794
  - Staging account: 545750752196
  - Production account: 720377205895
- GitHub repository access
- [Add other service-specific access requirements]

## Local Development Requirements

### Software Requirements
- Node.js 22.13.0 or higher
- npm 10.0.0 or higher
- AWS CLI v2+ configured with credentials
- AWS CDK CLI: `npm install -g aws-cdk`
- Git

### NPM Registry Configuration

#### For Local Development (Recommended)

Copy the local development configuration:
```bash
cp .npmrc-local .npmrc
```

This uses the public npm registry (https://registry.npmjs.org/) and requires **no authentication**.

#### For Production Deployment

Copy the production configuration:
```bash
cp .npmrc-prod .npmrc
```

Set the authentication token:
```bash
export PFG_ARTIFACTORY_NPM_AUTH="your-auth-token"
```

**Important Security Notes**:
- Never commit `.npmrc` with credentials to version control
- The `.npmrc` file is gitignored for security
- Use `.npmrc-local` for local development (no credentials needed)
- Use `.npmrc-prod` only when deploying to production

### AWS Configuration
```bash
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Default region: us-east-1
# Default output format: json
```

## Setup Steps

1. **Clone the repository**
   ```bash
   git clone [repository-url]
   cd service-agreement-reconciliation-pre-processor
   ```

2. **Configure NPM registry for local development**
   ```bash
   cp .npmrc-local .npmrc
   ```

3. **Install dependencies**
   ```bash
   npm install
   ```

4. **Configure AWS credentials**
   ```bash
   aws configure
   ```

5. **Bootstrap CDK (first time only)**
   ```bash
   cdk bootstrap aws://[account-id]/[region]
   ```

6. **Synthesize CloudFormation templates**
   ```bash
   npm run cdk synth
   ```

7. **Run tests**
   ```bash
   npm test
   ```

## Verification

After setup, verify everything works:

```bash
# Build the project
npm run build

# Run linter
npm run lint

# Synthesize CDK
npm run cdk synth

# Run tests
npm test
```

If all commands succeed, your environment is ready!

## Troubleshooting

### NPM Install Fails with Authentication Error

If you see authentication errors during `npm install`:
1. Verify you're using `.npmrc-local`: `cat .npmrc`
2. Should show `registry=https://registry.npmjs.org/`
3. If not, run: `cp .npmrc-local .npmrc`
4. Try again: `npm install`

### Cannot Access Private Packages

If you need access to private packages:
1. Use production configuration: `cp .npmrc-prod .npmrc`
2. Set authentication token: `export PFG_ARTIFACTORY_NPM_AUTH="your-token"`
3. Contact DevOps team for token if needed
```

### NETWORK.md Template

```markdown
# Network Configuration

## VPC Configuration

### Development Environment
- VPC ID: [to be configured]
- CIDR Block: [to be configured]
- Subnets:
  - Private Subnet 1: [to be configured]
  - Private Subnet 2: [to be configured]
  - Public Subnet 1: [to be configured]
  - Public Subnet 2: [to be configured]

### Staging Environment
- VPC ID: [to be configured]
- CIDR Block: [to be configured]
- Subnets: [to be configured]

### Production Environment
- VPC ID: [to be configured]
- CIDR Block: [to be configured]
- Subnets: [to be configured]

## Lambda Network Configuration

### VPC Enabled
- [Yes/No]

### Subnet Configuration
- Subnet Type: [Private/Public]
- Availability Zones: [List AZs]

### Security Groups
- Security Group ID: [to be configured]
- Ingress Rules:
  - [Rule 1]
  - [Rule 2]
- Egress Rules:
  - [Rule 1]
  - [Rule 2]

## External Connections

### AWS Services
- S3: [bucket names, regions]
- DynamoDB: [table names]
- [Other AWS services]

### External APIs
- [API name]: [endpoint, authentication]

### Database Connections
- [Database type]: [connection details]

## Firewall Rules

### Ingress
- [Source] → [Destination] : [Port] : [Protocol] : [Purpose]

### Egress
- [Source] → [Destination] : [Port] : [Protocol] : [Purpose]

## DNS Configuration

- [DNS records if applicable]

## Monitoring and Logging

- CloudWatch Log Groups: [log group names]
- VPC Flow Logs: [enabled/disabled]
```

### CONTRIBUTING.md Template

```markdown
# Contributing Guidelines

## Development Workflow

### 1. Create Feature Branch
```bash
git checkout -b feature/your-feature-name
```

### 2. Configure NPM for Local Development
```bash
# Use local development registry (public npm, no auth)
cp .npmrc-local .npmrc
```

**Important**: Always use `.npmrc-local` for local development to avoid authentication issues.

### 3. Make Changes
- Write code following TypeScript best practices
- Add tests for new functionality
- Update documentation as needed

### 4. Run Quality Checks
```bash
# Run linter
npm run lint

# Run tests
npm test

# Build project
npm run build

# Synthesize CDK
npm run cdk synth
```

### 5. Commit Changes
```bash
git add .
git commit -m "Description of changes"
```

Note: Pre-commit hooks will automatically run linting

### 6. Push and Create Pull Request
```bash
git push origin feature/your-feature-name
```

Then create a pull request on GitHub

## NPM Registry Configuration

### Local Development (Default)
- **File**: `.npmrc-local`
- **Registry**: https://registry.npmjs.org/ (public)
- **Authentication**: None required
- **Usage**: `cp .npmrc-local .npmrc`

### Production Deployment
- **File**: `.npmrc-prod`
- **Registry**: https://packages.principal.com/artifactory/api/npm/pfg-npm-dev-virt/
- **Authentication**: Required (PFG_ARTIFACTORY_NPM_AUTH)
- **Usage**: `cp .npmrc-prod .npmrc` + set auth token

### Security Rules
- ❌ **Never commit** `.npmrc` with credentials
- ✅ `.npmrc` is gitignored for security
- ✅ Template files (`.npmrc-local`, `.npmrc-prod`) are safe to commit
- ✅ Always use `.npmrc-local` for local development

## Code Standards

### TypeScript
- Use strict TypeScript configuration
- Define interfaces for all data structures
- Use meaningful variable and function names
- Add JSDoc comments for public APIs

### Testing
- Write unit tests for all new functions
- Aim for >80% code coverage
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)

### CDK Infrastructure
- Use CDK constructs appropriately
- Follow AWS best practices
- Add comments for complex infrastructure
- Use environment-specific configuration

## Adding New Lambda Functions

### 1. Create Function Directory
```bash
mkdir -p application/[function-name]/src
mkdir -p application/[function-name]/tests
```

### 2. Create package.json
```json
{
  "name": "[function-name]",
  "version": "1.0.0",
  "main": "src/index.ts",
  "scripts": {
    "test": "jest",
    "build": "tsc"
  },
  "dependencies": {},
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  }
}
```

### 3. Implement Handler
Create `application/[function-name]/src/index.ts`:
```typescript
import { Context } from 'aws-lambda';

export async function handler(event: any, context: Context) {
  // Implementation
}
```

### 4. Add Tests
Create `application/[function-name]/tests/index.test.ts`:
```typescript
import { handler } from '../src/index';

describe('[function-name]', () => {
  it('should handle events correctly', async () => {
    // Test implementation
  });
});
```

### 5. Update Infrastructure
Add Lambda function to infrastructure stack

## Adding New Infrastructure Constructs

### 1. Create Construct File
Create file in `infrastructure/lib/stacks/` or `infrastructure/lib/constructs/`

### 2. Define Construct
```typescript
import { Construct } from 'constructs';
import * as cdk from 'aws-cdk-lib';

export class MyConstruct extends Construct {
  constructor(scope: Construct, id: string, props?: MyConstructProps) {
    super(scope, id);
    // Implementation
  }
}
```

### 3. Add Tests
Create test in `infrastructure/test/`

### 4. Use in Stack
Import and use in appropriate stack

## Team Contacts

- **Team Lead**: [Name] - [email]
- **DevOps**: [Name] - [email]
- **Architecture**: [Name] - [email]

## Questions?

If you have questions, reach out to the team lead or create an issue on GitHub.
```

---

## Conclusion

This design provides a comprehensive approach to restructuring the service-agreement-reconciliation-pre-processor project to align with the risbi-data-pipeline organizational structure. The restructuring maintains all existing functionality while improving maintainability, documentation, and scalability.

Key benefits of this restructuring:
- Clear separation of concerns between application and infrastructure
- Comprehensive, specialized documentation
- Scalable structure for adding new Lambda functions
- Organized helper utilities for code reuse
- Type-safe configuration management
- Preserved TypeScript tooling advantages

The phased implementation approach with incremental verification ensures that issues are caught early and the project remains functional throughout the restructuring process.
