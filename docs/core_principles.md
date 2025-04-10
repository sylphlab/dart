# Dart Core Principles

These principles form the foundation of our Dart development practices.

## 1. Strong Typing First

- Fully leverage Dart's sound type system for code correctness and maintainability.
- Avoid using the `dynamic` type unless absolutely necessary and clearly justified (e.g., interacting with untyped external data). Prefer `Object?` or specific types.
- Use explicit type declarations for all public APIs (function parameters, return types, public class members). Type inference is acceptable for local variables where the type is obvious.
- Utilize generics (`<T>`) effectively to create reusable and type-safe components and functions.

## 2. Null Safety

- Comprehensively adopt and enforce Dart's sound null safety features. Code MUST be null-safe.
- Minimize the use of `late` variables. Prefer initializing fields directly in the constructor or using nullable types (`?`) if initialization might be delayed or absent. Use `late final` for lazy initialization where appropriate.
- Use nullable types (`Type?`) only when a value can genuinely be absent or represent a "no value" state. Avoid using nullable types just to bypass initialization requirements.
- Implement appropriate null checks (`if (x != null)`, `?.`, `??`) at API boundaries and when dealing with potentially null values.
- Strictly minimize the use of the non-null assertion operator (`!`). Its use requires strong justification and indicates a potential design flaw or unsafe assumption. Prefer null-aware operators or explicit checks.

## 3. Functional Programming Style

- **Immutability**: Prefer immutable data structures and classes. Use `final` for fields where possible. Utilize packages like `package:fast_immutable_collections` for immutable collections. For custom classes, implement `copyWith` methods (e.g., using `package:copy_with_extension_gen`).
- **Pure Functions**: Strive to write pure functions (functions whose output depends only on their input and have no side effects) whenever feasible, especially for business logic and data transformations.
- **Higher-Order Functions**: Utilize Dart's built-in higher-order functions (`map`, `where`, `fold`, `forEach`, etc.) for expressive and concise collection manipulation.
- **Avoid Side Effects**: Minimize functions that modify external state. Clearly document any necessary side effects.

## 4. Recommended Libraries & Patterns

*(Note: This list is not exhaustive and may evolve. Evaluate dependencies carefully.)*

- **State Management / Dependency Injection**:
    - **Preferred**: `riverpod` (using the annotation/generator style with `riverpod_generator`). Provides compile-time safety and excellent testability.
- **Immutable Collections**:
    - **Preferred**: `package:fast_immutable_collections`. Offers high-performance, truly immutable lists, sets, and maps.
- **Model Generation / Boilerplate Reduction**:
    - **JSON Serialization**: `package:json_serializable` (with `build_runner`). Generates `fromJson`/`toJson` methods. Configure `build.yaml` for options like `explicit_to_json: true`.
    - **Equality & `hashCode`**: Include `equatable: true` in `json_serializable`'s `build.yaml` configuration or use `package:equatable` directly if not using `json_serializable`.
    - **`copyWith`**: `package:copy_with_extension` (with `build_runner`). Generates `copyWith` methods for immutable classes.
    - **AVOID**: `package:freezed`. While powerful, its complexity and potential for large generated code often outweigh its benefits for our use cases. Prefer the combination of `json_serializable`, `equatable`, and `copy_with_extension`.
- **Error Handling**:
    - **Preferred**: Result types (e.g., `package:multiple_result`). Use `Result<SuccessType, ErrorType>` to explicitly handle success and failure cases instead of relying solely on exceptions for flow control.
- **Flutter Specific (If Applicable)**:
    - **Widget Logic/State**: `package:flutter_hooks` (reduces `StatefulWidget` boilerplate). Combine with Riverpod for state access.
    - **Animations**: `package:flutter_animate` (declarative and powerful animations).