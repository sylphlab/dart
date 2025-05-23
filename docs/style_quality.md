# Dart Code Style & Quality

This section outlines standards for code formatting, naming conventions, documentation, and quality enforcement.

## 1. Code Formatting

- **Tool**: `dart format`. This is the official Dart code formatter.
- **Enforcement**: Formatting MUST be enforced automatically.
    - **IDE**: Configure your IDE (VS Code, IntelliJ) to format on save.
    - **CI**: Include a formatting check step in the CI pipeline (`dart format . --output=none --set-exit-if-changed`).
    - **Pre-commit Hook (Optional but Recommended)**: Use a tool like `husky` (if in a mixed environment with Node.js) or a simple Git hook to run `dart format` before committing.
- **Line Length**: Maximum 80 characters. This improves readability, especially with side-by-side diffs.
- **Trailing Commas**: Always use trailing commas in parameter lists, argument lists, collection literals, etc., where allowed. This makes Git diffs cleaner when adding or reordering items.

## 2. Code Organization

- **Imports**:
    - Organize imports into three groups, separated by a blank line:
        1. `dart:*` imports.
        2. `package:*` imports.
        3. Relative imports (`import 'src/...'`).
    - Sort imports alphabetically within each group.
    - Use relative imports (`import 'src/...'`) for files within the same package's `lib` directory. Avoid `package:` imports for internal files.
- **Class Member Ordering**:
    - Follow a consistent logical order, for example:
        1. Static fields (`static final`, `static const`).
        2. Static methods.
        3. Instance fields (`final`, non-`final`).
        4. Constructors (primary, named, factory).
        5. Getters and setters.
        6. Instance methods.
    - Group related members together.
- **Visibility Ordering**:
    - Order declarations by visibility: public members first, then protected (`@protected` from `package:meta`), then private (`_`).

## 3. Naming Conventions

Adhere strictly to the [Effective Dart: Style - Naming](https://dart.dev/effective-dart/style#naming) guidelines:

- **Classes, Enums, Typedefs, Extensions, Type Parameters**: `UpperCamelCase`.
- **Libraries, Packages, Directories, Source Files**: `lowercase_with_underscores` (also known as `snake_case`).
- **Variables, Constants, Parameters, Prefixes, Methods, Functions**: `lowerCamelCase`.
- **Private Members**: Prepend with an underscore (`_lowerCamelCase`, `_UpperCamelCase` for private classes/enums).
- **Constants (`static const`, top-level `const`)**: `lowerCamelCase`. (Note: `SCREAMING_CAPS` is generally discouraged in Dart).
- **Acronyms/Abbreviations**: Treat them like regular words (e.g., `HttpRequest`, `imageUrl`, `utf8`). Capitalize only the first letter unless it's the first word (e.g., `DB`).

## 4. Documentation Comments

- **Tool**: `dartdoc`.
- **Requirement**: All public APIs (top-level variables/functions, public class members, constructors, enums, typedefs, extensions) MUST have documentation comments (`///`).
- **Content**:
    - Start with a single-sentence summary.
    - Add more detailed paragraphs as needed.
    - Use Markdown for formatting (e.g., code blocks `[]`, links).
    - Document parameters (`@param name description`), return values (`@return description`), and exceptions (`@throws ExceptionType description`).
    - Provide clear usage examples using Markdown code blocks, especially for complex APIs.
    - Link to related APIs using square brackets (`[memberName]`, `[ClassName.memberName]`).
- **Style**: Use `///` for dartdoc comments. Use `//` for implementation comments explaining *why* code does something, not *what* it does.

## 5. Code Quality Enforcement

- **Static Analysis**:
    - **Tool**: `dart analyze`.
    - **Configuration**: Code quality rules, including style, best practices, complexity limits, and code metrics, are defined and enforced through the `analysis_options.yaml` file.
    - **Shared Rules**: Projects MUST include the shared lint configuration package (`package:sylph_lints`) in their `analysis_options.yaml` to ensure consistency. See the [Setup & Config](setup_config.md) guide for details on including and potentially overriding rules.
    - **Additional Metrics**: Consider adding `package:dart_code_metrics` as a dev dependency and configuring its rules in `analysis_options.yaml` for deeper analysis (e.g., cyclomatic complexity, nesting depth, lines of code limits).
    - **CI Enforcement**: The CI pipeline MUST run `dart analyze --fatal-infos` to treat all diagnostics (including info and warnings) as errors, ensuring all reported issues are addressed.
- **Code Coverage**:
    - **Tool**: `package:coverage` and `package:test`.
    - **Goal**: Aim for high code coverage (e.g., >90-95% overall, potentially 100% for critical logic). Define specific goals per project/package.
    - **Enforcement**: Generate coverage reports (e.g., LCOV) in CI and use tools like Codecov to track coverage and potentially fail builds if coverage drops below thresholds. See the [Testing Guidelines](testing.md) for generation commands.

By leveraging `dart format` and enforcing strict analysis via `analysis_options.yaml` (including `package:sylph_lints`), we maintain a high standard of code quality and consistency across Dart projects.