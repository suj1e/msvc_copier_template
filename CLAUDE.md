# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Copier template** for generating Domain-Driven Design (DDD) microservices based on Spring Boot and the Cox platform framework. It generates a multi-module Maven project with a clean DDD architecture.

## Template Usage

### Generating a New Project

```bash
# Install copier if not already installed
pip install copier

# Generate a new project from this template
copier copy . /path/to/destination

# Or update an existing project
copier update /path/to/existing-project
```

The template will prompt for configuration values defined in `template/copier.yml`:
- `app_name`: Application name
- `group_id`: Maven group ID
- `artifact_id`: Maven artifact ID
- `app_version`: Application version
- `app_port`: Application port
- Feature flags: `enable_jpa`, `enable_es`, `enable_redis`, `enable_kafka`, `enable_bizlog`, `enable_cloud`, `enable_dynamictp`

### Generated Project Commands

After generating a project, use these commands in the generated project directory:

```bash
# Compile the project
sh compile.sh -c

# Install to local Maven repository
sh compile.sh -i

# Run tests with coverage report
sh compile.sh -t

# Deploy to remote repository
sh compile.sh -d
```

Note: The generated project uses `mvnd` (Maven Daemon) for faster builds.

## Architecture

### DDD Module Structure

The template generates a multi-module Maven project following DDD principles with clear separation of concerns:

```
{artifact_id}/
├── {artifact_id}-api/              # Public API contracts (DTOs, interfaces)
├── {artifact_id}-domain/           # Domain layer (entities, value objects, domain services)
├── {artifact_id}-application/      # Application layer (use cases, application services)
├── {artifact_id}-infrastructure/   # Infrastructure layer (repository implementations)
├── {artifact_id}-presentation/     # Presentation layer (REST controllers)
├── {artifact_id}-dubbo-provider/   # Dubbo RPC provider implementations
├── {artifact_id}-startup/          # Application startup and configuration
├── {artifact_id}-distribution/     # Packaging and deployment artifacts
└── {artifact_id}-tests/            # Integration and end-to-end tests
```

### Module Dependencies

The dependency flow follows DDD principles (inner layers don't depend on outer layers):

- **api**: No dependencies (pure contracts)
- **domain**: Depends on conditional features (JPA, ES, Redis, Kafka)
- **application**: Depends on domain
- **infrastructure**: Depends on domain (implements domain interfaces)
- **presentation**: Depends on application
- **dubbo-provider**: Depends on application
- **startup**: Depends on presentation, dubbo-provider, infrastructure (assembles everything)

### Key Technologies

- **Java 21**: Target Java version
- **Spring Boot**: Application framework
- **Cox Platform** (`org.flooc.cox:cox-platform`): Parent platform providing dependency management
- **Lombok**: Reduces boilerplate code
- **MapStruct**: Object mapping (note: must be configured AFTER Lombok in annotation processors)
- **Maven Daemon (mvnd)**: Fast Maven builds
- **Optional integrations**: JPA/QueryDSL, Elasticsearch, Redis/Redisson, Kafka, Nacos (cloud mode)

## Template Development

### File Structure

- `template/`: Contains all Jinja2 templates (`.jinja` suffix)
- `template/copier.yml`: Template configuration and questions
- Template variables use `{{variable_name}}` syntax
- Conditional blocks use `{% if condition %}...{% endif %}`

### Important Template Patterns

1. **Dynamic module names**: Directories use `{{artifact_id}}-*` pattern
2. **Conditional dependencies**: Features like JPA, Redis, Kafka are conditionally included based on flags
3. **Annotation processor ordering**: Lombok MUST come before MapStruct in `annotationProcessorPaths`
4. **Maven profiles**: Support for multiple environments (dev, test, staging, gray, prod) and cloud variants

### Modifying the Template

When editing template files:

1. **POM files**: Located at `template/pom.xml.jinja` and `template/{{artifact_id}}-*/pom.xml.jinja`
2. **Configuration**: Application YAML files in `template/{{artifact_id}}-startup/src/main/resources/`
3. **Scripts**: Build scripts at `template/compile.sh.jinja` and `template/flow.sh.jinja`
4. **Feature flags**: Add new flags in `template/copier.yml` and use them in templates with Jinja2 conditionals

### Testing Template Changes

```bash
# Generate a test project to verify changes
copier copy . /tmp/test-project

# Navigate to generated project and test build
cd /tmp/test-project/test-project
sh compile.sh -c
```

## Maven Build System

### Build Profiles

**Environment Profiles** (non-cloud):
- `dev-env`: Development (default when cloud disabled)
- `test-env`: Testing
- `staging-env`: Staging
- `gray-env`: Gray release
- `prod-env`: Production

**Cloud Profiles** (with Nacos):
- `cloud-dev-env`: Cloud development (default when cloud enabled)
- `cloud-test-env`: Cloud testing
- `cloud-staging-env`: Cloud staging
- `cloud-gray-env`: Cloud gray release
- `cloud-prod-env`: Cloud production

**Special Profiles**:
- `surefire-test`: Runs tests with JaCoCo coverage
- `git-commit-id-package`: Includes git commit info in build

### Maven Properties

- `revision`: Project version (supports CI/CD version management)
- `maven.deploy.skip`: Controls deployment (varies by module)
- `maven.install.skip`: Controls local installation (varies by module)
- `maven.test.skip`: Controls test execution (varies by module)

## Development Workflow

### Working on Template Files

1. Edit Jinja2 templates in `template/` directory
2. Update `template/copier.yml` if adding new configuration options
3. Test generation with `copier copy` to a temporary directory
4. Verify the generated project builds successfully

### Adding New Features

1. Add feature flag to `template/copier.yml` (e.g., `enable_new_feature`)
2. Add conditional dependencies in relevant `pom.xml.jinja` files
3. Add conditional configuration in application YAML templates
4. Update this CLAUDE.md if the feature significantly changes architecture

### Version Management

The template uses `${revision}` property for version management, allowing CI/CD systems to override versions during builds without modifying POM files.
