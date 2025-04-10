# Dart CI/CD Guidelines

Continuous Integration (CI) and Continuous Deployment (CD) are essential for maintaining code quality and automating releases. This guide outlines best practices using GitHub Actions.

## 1. CI Pipeline (GitHub Actions)

A typical CI pipeline for a Dart package or application should include the following stages, run on pushes to `main` and all pull requests:

- **Setup**: Check out code, set up Dart/Flutter environment.
- **Install Dependencies**: Fetch dependencies using `dart pub get`.
- **Format Check**: Verify code formatting using `dart format`.
- **Analyze**: Run static analysis using `dart analyze`.
- **Test**: Execute tests using `dart test`.
- **Coverage**: Generate and upload code coverage reports.

### Example GitHub Actions Workflow (`.github/workflows/ci.yml`)

```yaml
name: Dart CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest # Or windows-latest, macos-latest

    steps:
      - uses: actions/checkout@v4

      # Setup Dart SDK. Use dart-lang/setup-dart action
      # Pin the SDK version for consistency
      - name: Setup Dart SDK
        uses: dart-lang/setup-dart@v1
        with:
          sdk: '3.3.0' # Use the same major.minor as in pubspec.yaml

      # Optional: Setup Flutter (if needed)
      # - name: Setup Flutter SDK
      #   uses: subosito/flutter-action@v2
      #   with:
      #     flutter-version: '3.19.0' # Use the same major.minor as project
      #     channel: 'stable'
      #     cache: true

      # Optional: Setup Melos (if using a monorepo)
      - name: Activate Melos
        run: dart pub global activate melos ^6.0.0 # Use desired version

      # Install dependencies (using Melos or pub)
      - name: Install Dependencies (Melos)
        # If using Melos, bootstrap handles pub get in all packages
        run: melos bootstrap
        # Or if not using Melos:
        # run: dart pub get

      # Verify formatting
      - name: Verify Formatting (Melos)
        run: melos run format # Assumes 'format' script in melos.yaml
        # Or if not using Melos:
        # run: dart format . --output=none --set-exit-if-changed

      # Analyze project code
      - name: Analyze Project (Melos)
        run: melos run analyze # Assumes 'analyze' script in melos.yaml
        # Or if not using Melos:
        # run: dart analyze --fatal-infos

      # Run tests
      - name: Run Tests (Melos)
        run: melos run test # Assumes 'test' script in melos.yaml
        # Or if not using Melos:
        # run: dart test

      # Generate coverage report (LCOV)
      # Ensure coverage package is activated or use dart run
      - name: Generate Coverage Report
        run: |
          dart pub global activate coverage
          dart test --coverage=coverage
          dart pub global run coverage:format_coverage --lcov --in=coverage --out=coverage/lcov.info --packages=.dart_tool/package_config.json --report-on=lib
        # Note: Melos doesn't easily aggregate coverage yet. Run in root or per-package.

      # Upload coverage report to Codecov
      - name: Upload Coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }} # Store Codecov token in GitHub secrets
          files: coverage/lcov.info
          fail_ci_if_error: true
          verbose: true
```

**Key Considerations:**

- **SDK Version**: Pin the Dart (and Flutter, if used) SDK version in the `setup-dart` action to match the version range specified in `pubspec.yaml` for consistency.
- **Caching**: Use caching actions (like `actions/cache` or built-in caching in `setup-dart`/`flutter-action`) for `pub` dependencies to speed up workflows.
- **Melos**: If using Melos, define scripts in `melos.yaml` and call them using `melos run <script_name>`. `melos bootstrap` handles dependency fetching for all packages.
- **Secrets**: Store sensitive information like API keys (e.g., `CODECOV_TOKEN`) in GitHub repository secrets.
- **Error Handling**: Ensure steps fail the workflow on error (e.g., `--fatal-infos` for `analyze`, `--set-exit-if-changed` for `format`, `fail_ci_if_error: true` for Codecov).

## 2. CD Pipeline (Publishing Packages)

For packages intended for pub.dev, automate the publishing process.

- **Trigger**: Typically triggered on creating a GitHub release or merging to the `main` branch with specific version tags.
- **Versioning**: Use a tool or manual process to bump the version in `pubspec.yaml` according to SemVer before publishing. Consider conventional commits and tools that automate versioning based on commit messages.
- **Authentication**: Publishing requires authentication with pub.dev. Store credentials securely (e.g., using GitHub secrets). Refer to the [pub.dev publishing documentation](https://dart.dev/tools/pub/publishing#automated-publishing) for setting up credentials for CI.
- **Dry Run**: Always perform a dry run (`dart pub publish --dry-run`) before the actual publish step to catch potential issues.

### Example Publish Workflow Snippet (Triggered on Release)

```yaml
name: Publish Package

on:
  release:
    types: [ published ] # Trigger when a GitHub release is published

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write # Required for trusted publishing

    steps:
      - uses: actions/checkout@v4

      - name: Setup Dart SDK
        uses: dart-lang/setup-dart@v1
        with:
          sdk: 'stable' # Or pin version

      # Optional: Get version from tag/release
      # - name: Get the version
      #   id: get_version
      #   run: echo ::set-output name=VERSION::${GITHUB_REF#refs/tags/}

      # Optional: Update pubspec.yaml version (if not done before release)
      # - name: Update version
      #   run: dart pub pubspec set version ${{ steps.get_version.outputs.VERSION }}

      - name: Install Dependencies
        run: dart pub get

      - name: Check Publish Warnings
        run: dart pub publish --dry-run

      # Publish using trusted publishing (preferred)
      - name: Publish to pub.dev
        run: dart pub publish --force # Use --force only after dry run and checks
```

**Trusted Publishing (Recommended):**

Configure pub.dev to trust GitHub Actions to publish on your behalf without needing explicit secrets stored in GitHub. Follow the [Trusted Publishing setup guide](https://dart.dev/tools/pub/publishing#trusted-publishing-with-github-actions).

## 3. CD Pipeline (Deploying Applications)

For applications (e.g., Flutter web apps, server-side Dart), the CD pipeline will involve building the application and deploying it to the target environment (e.g., Firebase Hosting, Google Cloud Run, Docker registry).

- **Build**: Use `flutter build web`, `dart compile exe`, or Docker builds depending on the application type.
- **Deploy**: Use specific actions or CLI tools for the target platform (e.g., `FirebaseExtended/action-hosting-deploy@v0`, `gcloud run deploy`, `docker push`).
- **Environments**: Use GitHub Environments to manage deployment secrets and approval workflows for different stages (staging, production).