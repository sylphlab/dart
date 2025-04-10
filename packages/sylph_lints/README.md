# Sylph Lints

Shared lint rules for SylphLab Dart projects. This package provides a stricter set of rules on top of the standard `package:lints/recommended.yaml`.

## Installation

Add `sylph_lints` to your `dev_dependencies` in `pubspec.yaml`:

```yaml
dev_dependencies:
  sylph_lints: ^0.1.0 # Use the desired version
```

Then, include the rules in your project's `analysis_options.yaml`:

```yaml
include: package:sylph_lints/analysis_options.yaml

# You can add project-specific customizations below
# linter:
#   rules:
#     # Disable a specific rule from sylph_lints if needed
#     - avoid_print: false
```

## Rules

This package enables rules from `package:lints/recommended.yaml` plus additional rules focused on consistency, potential errors, and style preferences favored by SylphLab. Refer to the `analysis_options.yaml` in this package for the complete list.

## Contributing

See the main [Dart repository README](https://github.com/sylphlab/dart/blob/main/README.md).

## License

MIT License - see the [LICENSE](LICENSE) file for details.