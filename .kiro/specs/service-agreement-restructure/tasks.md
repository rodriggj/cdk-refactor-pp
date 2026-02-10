# Implementation Plan: Service Agreement Project Restructuring

## Overview

This implementation plan provides a step-by-step approach to restructure the service-agreement-reconciliation-pre-processor project to align with the risbi-data-pipeline organizational structure. The tasks are organized in phases, with each phase building on the previous one. The restructuring will be incremental and verifiable at each step.

## Tasks

- [ ] 1. Phase 1: Create Documentation Files
  - [ ] 1.1 Create PREREQS.md with prerequisites and setup instructions
    - Create file at root level
    - Include required access section (AWS accounts, GitHub)
    - Include local development requirements (Node.js, AWS CLI, CDK CLI)
    - Include NPM registry configuration section (local vs production)
    - Include setup steps (clone, configure npm, install, configure AWS, bootstrap, synth)
    - Include verification section
    - Include troubleshooting section for NPM authentication issues
    - _Requirements: 1.1, 1.5, 8.4, 8.8, 8.9, 13.4, 13.5_
  
  - [ ] 1.2 Create NETWORK.md with network configuration documentation
    - Create file at root level
    - Include VPC configuration section (placeholder for each environment)
    - Include Lambda network configuration section
    - Include external connections section
    - Include firewall rules section
    - _Requirements: 1.2, 1.6_
  
  - [ ] 1.3 Create CONTRIBUTING.md with contribution guidelines
    - Create file at root level
    - Include development workflow section with NPM registry setup
    - Include NPM registry configuration section (local vs production)
    - Include security rules for .npmrc files
    - Include code standards section (TypeScript, testing, CDK)
    - Include section on adding new Lambda functions with example
    - Include section on adding new infrastructure constructs
    - Include team contacts section (placeholder)
    - _Requirements: 1.3, 1.7, 9.2, 9.6, 9.7, 13.9_
  
  - [ ] 1.4 Update README.md to reference new documentation files
    - Add links to PREREQS.md, NETWORK.md, CONTRIBUTING.md
    - Add section referencing architecture diagrams (to be created)
    - Add note about NPM registry configuration for local development
    - Maintain existing content
    - _Requirements: 1.4, 2.5_
  
  - [ ] 1.5 Create NPM registry configuration files
    - Create .npmrc-local with public registry configuration (no auth)
    - Create .npmrc-prod with private registry configuration (requires auth)
    - Update .gitignore to include .npmrc (prevent credential commits)
    - Document the difference between local and production configurations
    - _Requirements: 13.1, 13.2, 13.3, 13.6, 13.7, 13.8_

- [ ] 2. Phase 2: Create Directory Structure
  - [ ] 2.1 Create application directory structure
    - Create application/ directory at root level
    - Create .gitkeep file to preserve empty directory
    - _Requirements: 3.1, 7.2_
  
  - [ ] 2.2 Create diagram directory structure
    - Create diagram/ directory at root level
    - Create diagram/README.md with instructions for creating and updating diagrams
    - Add placeholder for architecture.png
    - _Requirements: 2.1, 2.2, 2.4, 7.3_
  
  - [ ] 2.3 Create tests directory structure
    - Create tests/ directory at root level
    - Create tests/integration/ subdirectory
    - Create tests/e2e/ subdirectory
    - Create tests/helpers/ subdirectory for test utilities
    - Add .gitkeep files to preserve empty directories
    - _Requirements: 5.1, 5.2, 5.3, 7.5_
  
  - [ ] 2.4 Create infrastructure helpers directory
    - Create infrastructure/lib/helpers/ directory
    - Create infrastructure/lib/helpers/index.ts for barrel exports
    - _Requirements: 4.1, 4.3_
  
  - [ ]* 2.5 Verify directory structure
    - Run script to verify all directories exist
    - Verify infrastructure/test/ still exists
    - _Requirements: 5.4, 7.1, 11.2_

- [ ] 3. Checkpoint - Verify Structure
  - Ensure directory structure is correct
  - Run `cdk synth` to ensure no breakage
  - Ask user if questions arise

- [ ] 4. Phase 3: Create Enhanced Configuration
  - [ ] 4.1 Create config.ts with TypeScript interfaces
    - Create infrastructure/lib/config.ts
    - Define EnvironmentConfig interface
    - Define Tags interface
    - Define AppConfig interface
    - Define Configuration interface
    - _Requirements: 6.1, 6.2_
  
  - [ ] 4.2 Implement configuration object with all environments
    - Add app configuration with appName and tags
    - Add Development environment configuration
    - Add Staging environment configuration
    - Add Production environment configuration
    - Add DevelopmentDr environment configuration
    - Add StagingDr environment configuration
    - Add ProductionDr environment configuration
    - Migrate existing configuration from bin/app.ts
    - _Requirements: 6.3, 6.4, 6.5, 6.6_
  
  - [ ] 4.3 Add helper functions to config.ts
    - Implement getEnvironmentConfig() function
    - Implement getEnvironmentTags() function to merge app and environment tags
    - _Requirements: 6.4_
  
  - [ ] 4.4 Update bin/app.ts to use new configuration
    - Import config from lib/config.ts
    - Replace inline configuration with config imports
    - Use getEnvironmentConfig() and getEnvironmentTags() helpers
    - Remove duplicate configuration code
    - _Requirements: 6.1_
  
  - [ ] 4.5 Update @types/config.d.ts if needed
    - Ensure type definitions match new config structure
    - Remove obsolete type definitions
    - _Requirements: 6.7_
  
  - [ ]* 4.6 Verify configuration changes
    - Run TypeScript compilation: `npm run build`
    - Run `cdk synth` to verify configuration works
    - Verify no TypeScript errors
    - _Requirements: 6.7, 11.4_

- [ ] 5. Checkpoint - Verify Configuration
  - Ensure TypeScript compiles without errors
  - Ensure `cdk synth` succeeds
  - Ask user if questions arise

- [ ] 6. Phase 4: Create Helper Utilities (Optional - Create as Needed)
  - [ ] 6.1 Create IAM helper utilities (if needed)
    - Create infrastructure/lib/helpers/iam.ts
    - Define RoleConfig interface
    - Implement createLambdaRole() function
    - Add other IAM utility functions as needed
    - _Requirements: 4.2_
  
  - [ ] 6.2 Create KMS helper utilities (if needed)
    - Create infrastructure/lib/helpers/kms.ts
    - Define KmsKeyConfig interface
    - Implement createKmsKey() function
    - Add other KMS utility functions as needed
    - _Requirements: 4.2_
  
  - [ ] 6.3 Create networking helper utilities (if needed)
    - Create infrastructure/lib/helpers/networking.ts
    - Define VpcConfig interface
    - Implement getVpcFromConfig() function
    - Add other networking utility functions as needed
    - _Requirements: 4.2_
  
  - [ ] 6.4 Update helpers/index.ts with barrel exports
    - Export all helper modules
    - Ensure clean import paths for infrastructure code
    - _Requirements: 4.3, 4.7_
  
  - [ ]* 6.5 Verify helper utilities
    - Run TypeScript compilation
    - Verify helpers can be imported independently
    - Run `cdk synth` to ensure no breakage
    - _Requirements: 4.7, 11.3_

- [ ] 7. Phase 5: Application Layer Setup (If Lambda Code Exists)
  - [ ] 7.1 Identify existing Lambda function code
    - Search infrastructure code for Lambda function definitions
    - Identify any inline Lambda code
    - Document Lambda functions that need to be moved
    - _Requirements: 3.2_
  
  - [ ] 7.2 Create Lambda function directory structure
    - For each Lambda function, create application/[function-name]/ directory
    - Create application/[function-name]/src/ directory
    - Create application/[function-name]/tests/ directory
    - _Requirements: 3.2, 3.4_
  
  - [ ] 7.3 Create package.json for each Lambda function
    - Create application/[function-name]/package.json
    - Include function-specific dependencies
    - Include test scripts
    - Include build scripts
    - _Requirements: 3.3_
  
  - [ ] 7.4 Create tsconfig.json for each Lambda function
    - Create application/[function-name]/tsconfig.json
    - Configure TypeScript for Lambda runtime
    - Extend root tsconfig if appropriate
    - _Requirements: 3.3_
  
  - [ ] 7.5 Move Lambda handler code to application directory
    - Move Lambda handler code to application/[function-name]/src/index.ts
    - Move supporting modules to application/[function-name]/src/
    - Update import paths within Lambda code
    - _Requirements: 3.2_
  
  - [ ] 7.6 Move Lambda tests to application directory
    - Move Lambda tests to application/[function-name]/tests/
    - Update test import paths
    - _Requirements: 3.4, 5.5_
  
  - [ ] 7.7 Update infrastructure code to reference new Lambda locations
    - Update Lambda function constructs to reference application/[function-name]
    - Update handler paths
    - Update asset paths for Lambda code
    - _Requirements: 3.6, 10.3_
  
  - [ ]* 7.8 Verify Lambda code migration
    - Run TypeScript compilation for each Lambda function
    - Run Lambda function tests
    - Run infrastructure TypeScript compilation
    - Run `cdk synth` to verify Lambda references work
    - _Requirements: 10.2, 10.3, 11.3_

- [ ] 8. Checkpoint - Verify Application Layer
  - Ensure all Lambda code compiles
  - Ensure all Lambda tests pass
  - Ensure `cdk synth` succeeds
  - Ask user if questions arise

- [ ] 9. Phase 6: Testing Structure Organization
  - [ ] 9.1 Identify existing integration tests (if any)
    - Search for integration tests in infrastructure/test/
    - Identify tests that should be moved to root tests/
    - _Requirements: 5.6_
  
  - [ ] 9.2 Move integration tests to tests/integration/
    - Move identified integration tests to tests/integration/
    - Update import paths in tests
    - _Requirements: 5.6_
  
  - [ ] 9.3 Create test utilities (if needed)
    - Create tests/helpers/test-utils.ts
    - Implement common test helper functions
    - Export utilities for use in tests
    - _Requirements: 5.6_
  
  - [ ] 9.4 Update Jest configuration for new test locations
    - Update jest.config.json to include tests/ directory
    - Ensure infrastructure/test/ is still included
    - Ensure application/*/tests/ are included
    - Configure test coverage paths
    - _Requirements: 5.1, 5.4_
  
  - [ ]* 9.5 Verify test organization
    - Run `npm test` to verify all tests are discovered
    - Verify tests pass
    - Verify test coverage includes all directories
    - _Requirements: 8.7, 10.4_

- [ ] 10. Checkpoint - Verify Testing Structure
  - Ensure all tests are discovered and pass
  - Ensure test coverage is appropriate
  - Ask user if questions arise

- [ ] 11. Phase 7: Create Validation Tests
  - [ ] 11.1 Create restructuring validation unit tests
    - Create tests/integration/restructuring-validation.test.ts
    - Add tests for directory structure existence
    - Add tests for documentation file existence
    - Add tests for configuration structure
    - Add tests for tooling preservation (ESLint, Jest, Husky, VS Code)
    - _Requirements: 7.1, 7.6, 12.3, 12.4, 12.5_
  
  - [ ]* 11.2 Create property-based tests for Lambda structure
    - Install fast-check: `npm install --save-dev fast-check`
    - Create tests/integration/lambda-structure.property.test.ts
    - **Property 1: Lambda Function Structure Consistency**
    - **Validates: Requirements 3.3, 3.4**
  
  - [ ]* 11.3 Create property-based tests for infrastructure references
    - Create tests/integration/infrastructure-references.property.test.ts
    - **Property 3: Infrastructure References Application Code**
    - **Validates: Requirements 3.6, 10.3**
  
  - [ ]* 11.4 Create property-based tests for helper independence
    - Create tests/integration/helper-independence.property.test.ts
    - **Property 4: Helper Module Independence**
    - **Validates: Requirements 4.7, 9.3**
  
  - [ ]* 11.5 Create property-based tests for test location
    - Create tests/integration/test-location.property.test.ts
    - **Property 5: Application Test Location**
    - **Property 6: Integration Test Location**
    - **Validates: Requirements 5.5, 5.6**
  
  - [ ]* 11.6 Create property-based tests for environment configuration
    - Create tests/integration/environment-config.property.test.ts
    - **Property 7: Environment Configuration Completeness**
    - **Validates: Requirements 6.6**
  
  - [ ]* 11.7 Create property-based tests for Lambda independence
    - Create tests/integration/lambda-independence.property.test.ts
    - **Property 8: Lambda Function Independence**
    - **Validates: Requirements 9.1**
  
  - [ ]* 11.8 Create property-based tests for functionality preservation
    - Create tests/integration/functionality-preservation.property.test.ts
    - **Property 10: CDK Stack Functionality Preservation**
    - **Property 11: Lambda Function Functionality Preservation**
    - **Validates: Requirements 10.1, 10.2, 10.5, 10.6**

- [ ] 12. Phase 8: Final Verification and Documentation
  - [ ] 12.1 Run full test suite
    - Run `npm test` and verify all tests pass
    - Review test coverage
    - _Requirements: 8.7, 10.4_
  
  - [ ] 12.2 Run TypeScript compilation
    - Run `npm run build` and verify successful compilation
    - Verify no TypeScript errors
    - _Requirements: 8.6, 11.4_
  
  - [ ] 12.3 Run CDK synthesis
    - Run `cdk synth` and verify successful synthesis
    - Save synthesized templates for comparison
    - _Requirements: 8.2, 11.3, 11.5_
  
  - [ ] 12.4 Compare CDK output with pre-restructuring baseline (if available)
    - Compare synthesized CloudFormation templates
    - Verify functional equivalence
    - Document any differences
    - _Requirements: 10.6_
  
  - [ ] 12.5 Verify local instantiation without private registry
    - Remove any existing .npmrc: `rm .npmrc`
    - Copy local development config: `cp .npmrc-local .npmrc`
    - Run `npm install` in clean environment
    - Verify installation completes without authentication errors
    - Run `cdk synth` to verify project is instantiable
    - _Requirements: 8.1, 8.5, 8.8, 8.9, 13.5_
  
  - [ ] 12.6 Create architecture diagram
    - Create architecture diagram showing restructured project
    - Save as diagram/architecture.png
    - Update diagram/README.md with description
    - _Requirements: 2.2, 2.3_
  
  - [ ] 12.7 Update main README.md with restructuring notes
    - Document the new structure
    - Add links to all documentation
    - Add section on project organization
    - Add note about NPM registry configuration
    - _Requirements: 1.4_
  
  - [ ] 12.8 Verify NPM registry configuration
    - Verify .npmrc-local uses public registry
    - Verify .npmrc-prod uses private registry
    - Verify .npmrc is in .gitignore
    - Verify documentation explains both configurations
    - _Requirements: 13.1, 13.2, 13.3, 13.4, 13.8, 13.9_
  
  - [ ] 12.9 Verify all requirements are met
    - Review requirements document
    - Verify each requirement has been addressed
    - Document any deviations or notes
    - _Requirements: 11.2_

- [ ] 13. Final Checkpoint - Complete Verification
  - Ensure all tests pass
  - Ensure `cdk synth` succeeds
  - Ensure `cdk deploy` can be executed (if appropriate)
  - Review all documentation
  - Ask user if questions arise

## Notes

- Tasks marked with `*` are optional property-based tests that provide additional validation
- Each phase includes verification steps to catch issues early
- Checkpoints ensure incremental validation before proceeding
- Phase 5 (Application Layer) may be minimal if no Lambda code currently exists
- Phase 6 (Helper Utilities) creates helpers as needed, not all may be required initially
- The restructuring preserves all existing functionality while improving organization
- Git commits should be made after each successful phase for rollback capability
- If any verification fails, stop and resolve the issue before proceeding

## Success Criteria

The restructuring is complete when:
- All documentation files exist with required content
- All directories match the target structure
- Configuration is centralized in config.ts
- Helper utilities are organized in helpers/ directory
- Application code (if any) is in application/ directory
- Tests are organized by type (unit, integration, e2e)
- NPM registry configuration files exist (.npmrc-local, .npmrc-prod)
- .npmrc is gitignored to prevent credential leaks
- `npm install` completes without errors using public registry (local dev)
- `npm run build` completes without errors
- `npm test` runs and all tests pass
- `cdk synth` succeeds
- Project structure matches risbi-pipeline pattern
- TypeScript tooling (ESLint, Jest, Husky) still works
- VS Code configuration is preserved
- Local development works without private registry authentication
