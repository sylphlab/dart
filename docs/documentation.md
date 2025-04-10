# Dart Documentation Guidelines

Clear and comprehensive documentation is vital for maintainable code and usable libraries.

## 1. API Documentation (`dartdoc`)

- **Tool**: `dartdoc`. The official Dart tool for generating API documentation from source code comments.
- **Requirement**: All public APIs MUST have `///` documentation comments. This includes:
    - Top-level functions, variables, constants, typedefs.
    - Public classes, enums, extensions.
    - Public constructors, methods, fields, getters, setters within classes/enums/extensions.
- **Comment Content**:
    - Start with a concise single-sentence summary.
    - Follow with more detailed paragraphs explaining purpose, usage, and behavior.
    - Use Markdown for formatting (code blocks `[]`, lists, links).
    - Document parameters using `@param name description`.
    - Document return values using `@return description` (often inferred, but useful for clarity).
    - Document exceptions thrown using `@throws ExceptionType description`.
    - Provide clear usage examples within code blocks (` ```dart ... ``` `).
    - Link to other relevant APIs using square brackets (`[ClassName]`, `[methodName]`, `[ClassName.methodName]`).
- **Configuration (`dartdoc_options.yaml`)**:
    - Use `dartdoc_options.yaml` to customize generation (see [Setup & Config](setup_config.md)).
    - Define categories to organize the documentation.
    - Set error/warning levels for documentation issues (e.g., broken links, unresolved references).
    - Configure linking to the source code repository.
- **Generation**:
    - Run `dart doc` (or `dart pub global run dartdoc`) to generate documentation (usually outputs to `doc/api/`).
    - Integrate generation into CI/CD for automated deployment (e.g., to GitHub Pages).

## 2. Project Documentation (READMEs, Guides)

- **Package `README.md`**: Each package (e.g., `packages/sylph_lints`) MUST have a `README.md` explaining:
    - What the package does.
    - How to install it (`pubspec.yaml` snippet).
    - Basic usage examples.
    - Links to more detailed documentation (e.g., API docs, other guides).
    - Contribution guidelines (or link to main repo guidelines).
    - License information.
- **Repository Root `README.md`**: The main `README.md` for the monorepo (`configs/dart/README.md`) should:
    - Briefly describe the purpose of the repository (shared Dart configs/guidelines).
    - List the contained packages and link to their READMEs.
    - Provide links to the general guideline documents in `docs/`.
    - Include setup instructions if necessary (e.g., activating Melos).
- **Guideline Documents (`docs/`)**:
    - Organize guidelines into logical Markdown files within the `docs/` directory (as we are doing now).
    - Write clearly and concisely.
    - Use code examples to illustrate concepts.
    - Link between related guideline documents.

## 3. Implementation Comments (`//`)

- Use `//` for comments within method bodies or for private members.
- Explain *why* the code is doing something, not *what* it's doing (the code itself should explain the *what*).
- Clarify complex logic, workarounds, or non-obvious decisions.
- Use `// TODO:` comments to mark areas needing future attention. Include context or an issue tracker link if possible.
- Keep comments up-to-date with code changes. Remove outdated comments.