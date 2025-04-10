# Dart Technology Stack & Configuration

This section details the standard tools, versions, and configuration files used in SylphLab Dart projects.

## 1. Core Stack

- **Runtime**: Dart SDK. **MUST use the latest stable version**. Pin the version in CI environments for reproducibility. Check the [Dart SDK archive](https://dart.dev/get-dart/archive) for versions.
- **Flutter**: If used, **MUST use the latest stable version**. Pin the version in CI. Check the [Flutter releases](https://docs.flutter.dev/release/archive) page.
- **Package Manager**: `dart pub` (comes with the Dart SDK). **MUST use the latest version** bundled with the chosen SDK. Use `dart pub upgrade --major-versions` regularly to keep dependencies up-to-date. Use `dart pub outdated` to check for outdated dependencies.

## 2. Standard Configuration Files

### `analysis_options.yaml`

This file configures static analysis (lints and hints).

- **In Packages (e.g., `packages/sylph_lints`)**: Defines the rules the package exports.
  ```yaml
  # packages/sylph_lints/analysis_options.yaml
  analyzer:
    language:
      strict-casts: true
      strict-raw-types: true
    errors:
      todo: warning # Allow TODOs
      dead_code: error
      # Add other enforced errors

  linter:
    rules:
      # List all exported rules
      - prefer_final_locals
      - prefer_const_constructors
      # ... (see full list in the package)
  ```

- **In Consuming Projects**: Includes the shared rules and adds project-specific overrides.
  ```yaml
  # project_root/analysis_options.yaml
  include: package:sylph_lints/analysis_options.yaml # Include shared rules

  analyzer:
    exclude:
      - "**/*.g.dart" # Exclude generated files
      - "**/*.freezed.dart"
      # Add project-specific excludes
    errors:
      # Override specific rule severities if absolutely necessary (with justification)
      # unnecessary_lambdas: ignore

  linter:
    rules:
      # Enable project-specific rules not in the shared set
      # - avoid_print # Example: Discourage print statements in this project
      # Disable a rule from the shared set if needed (provide reasoning)
      # - public_member_api_docs: false
  ```
- **Code Metrics**: Consider adding `package:dart_code_metrics` as a dev dependency and configuring its rules in `analysis_options.yaml` for deeper analysis (complexity, anti-patterns).

### `pubspec.yaml`

Defines package metadata, dependencies, and tool configurations.

- **Metadata**:
    - `name`: `snake_case` for packages intended for pub.dev or internal sharing.
    - `description`: Clear, concise description.
    - `version`: Follow [Semantic Versioning](https://semver.org/). Use `0.y.z` for initial development.
    - `repository`: Link to the primary source code repository (e.g., GitHub).
    - `homepage`: Link to documentation or project website (optional but recommended).
    - `publish_to: none`: Use for projects/packages not intended for pub.dev (e.g., example apps, root of monorepo).
- **Environment**:
    - `sdk`: Specify a tight constraint using the latest stable Dart SDK version (e.g., `'>=3.3.0 <4.0.0'`). Avoid overly broad ranges.
- **Dependencies**:
    - List only direct runtime dependencies.
    - Use version constraints carefully. Prefer caret syntax (`^`) for stable dependencies following SemVer. Pin exact versions (`x.y.z`) only if necessary.
- **Dev Dependencies**:
    - Include tools for testing, linting, code generation, etc.
    - `lints`: Use the standard `lints` package (e.g., `^3.0.0`). Our custom rules build upon this.
    - `test`: Standard testing framework.
    - `build_runner`, `json_serializable`, `copy_with_extension`, etc., if using code generation.
    - `melos`: If managing a monorepo.
- **Platforms (Optional)**:
    - For library packages, explicitly declare supported platforms if not all are supported.

### `dart_test.yaml` (Optional)

Configures the test runner (`package:test`).

- Define presets for different test types (unit, integration).
- Set timeouts.
- Define and use tags for filtering tests (e.g., `@integration`, `@slow`).
- Specify platforms if needed (e.g., `vm`, `chrome`).

```yaml
# dart_test.yaml (Example)
presets:
  unit:
    timeout: 30s
  integration:
    timeout: 2m # Longer timeout for integration tests
    tags: integration

tags:
  integration: # Allows running integration tests via `dart test -t integration`
  slow:

platforms:
  - vm # Default platform
  # - chrome
```

### `dartdoc_options.yaml` (Optional)

Configures the API documentation generator (`dartdoc`).

- Define categories for better organization.
- Set error/warning levels for documentation issues.
- Configure linking to source code.

```yaml
# dartdoc_options.yaml (Example)
dartdoc:
  categories:
    "Core":
      markdown: doc/categories/core.md # Link to overview markdown
    "Utilities":
      markdown: doc/categories/utilities.md
  categoryOrder: ["Core", "Utilities"]
  errors: # Fail generation on these issues
    - unresolved-doc-reference
    - duplicate-file
  warnings: # Warn about these issues
    - ambiguous-doc-reference
  linkToSource:
    root: '.' # Relative to project root
    uriTemplate: 'https://github.com/sylphlab/dart/blob/main/%f%#L%l%' # Link to GitHub
```

### `melos.yaml` (For Monorepos)

Configures the Melos tool for managing Dart monorepos.

- `name`: Monorepo name.
- `packages`: Glob pattern for locating packages (e.g., `packages/**`).
- `scripts`: Define common tasks runnable via `melos run <script_name>` (e.g., `analyze`, `format`, `test`, `bootstrap`). Use `melos exec` to run commands within each package.

```yaml
# melos.yaml (Example)
name: sylph_dart_monorepo

packages:
  - packages/**
  # - example/** # Include example apps if needed

scripts:
  bootstrap:
    run: melos exec -- dart pub get
    description: Install dependencies in all packages.
  analyze:
    run: melos exec -- dart analyze . --fatal-infos
    description: Analyze all packages.
  format:
    run: melos exec -- dart format . --set-exit-if-changed
    description: Format code in all packages.
  test:
    run: melos exec -- dart test
    description: Run tests in all packages.
```

## 3. Dependency Management

- **Regular Updates**: Use `dart pub outdated` frequently to identify outdated dependencies.
- **Major Version Upgrades**: Use `dart pub upgrade --major-versions` cautiously. Review breaking changes before upgrading major versions.
- **Security**: Regularly check for known vulnerabilities in dependencies. Consider using automated tools or services if available for Dart.
- **`pubspec.lock`**: Always commit the `pubspec.lock` file to ensure reproducible builds across different environments. CI should typically use `dart pub get --offline` or rely on the lock file implicitly.