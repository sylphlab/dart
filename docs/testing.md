# Dart Testing Guidelines

Robust testing is crucial for building reliable Dart applications. This section covers testing frameworks, best practices, and coverage reporting.

## 1. Testing Framework

- **Primary Framework**: `package:test`. This is the standard, feature-rich testing framework for Dart.
- **Mocking**: `package:mockito`. A popular library for creating mock objects, essential for isolating units during testing. Use code generation (`build_runner`) with Mockito for type safety.
- **Matchers**: Utilize the extensive matchers provided by `package:test` (e.g., `expect`, `equals`, `throwsA`, `isA`, `contains`).

## 2. Test Organization

- **Directory Structure**: Place tests in the `test/` directory at the root of the package or project. Mirror the structure of the `lib/` directory.
  ```
  /
  ├── lib/
  │   ├── src/
  │   │   ├── services/
  │   │   │   └── auth_service.dart
  │   │   └── utils/
  │   │       └── string_formatter.dart
  │   └── my_package.dart
  ├── test/
  │   ├── src/
  │   │   ├── services/
  │   │   │   └── auth_service_test.dart
  │   │   └── utils/
  │   │       └── string_formatter_test.dart
  │   └── my_package_test.dart # Optional: Test main library exports
  └── ...
  ```
- **File Naming**: Test files MUST end with `_test.dart`.
- **Grouping**: Use `group()` function within test files to organize related tests logically. Nest groups as needed.

## 3. Testing Best Practices

- **AAA Pattern**: Structure tests using Arrange, Act, Assert:
    - **Arrange**: Set up the necessary preconditions and inputs (e.g., create object instances, mocks).
    - **Act**: Execute the code being tested.
    - **Assert**: Verify the outcome using `expect()` and appropriate matchers.
- **Test Isolation**: Each test should be independent and not rely on the state or outcome of other tests. Use `setUp()` and `tearDown()` functions (provided by `package:test`) for setting up and cleaning up resources shared *within a group* of tests.
  ```dart
  group('AuthService', () {
    late MockHttpClient mockHttpClient;
    late AuthService authService;

    setUp(() {
      // Arrange: Runs before each test in this group
      mockHttpClient = MockHttpClient();
      authService = AuthService(mockHttpClient);
    });

    test('login succeeds with valid credentials', () async {
      // Arrange (specific to this test)
      when(mockHttpClient.post(any, body: anyNamed('body')))
          .thenAnswer((_) async => Response('{"token": "abc"}', 200));

      // Act
      final result = await authService.login('user', 'pass');

      // Assert
      expect(result, isTrue);
      verify(mockHttpClient.post(Uri.parse('/login'), body: anyNamed('body'))).called(1);
    });

    // More tests...
  });
  ```
- **Descriptive Names**: Test names (the string passed to `test()`) should clearly describe the scenario being tested.
- **Focus on Public API**: Primarily test the public API of your classes and libraries. Testing private implementation details makes tests brittle.
- **Test Edge Cases**: Include tests for boundary conditions, invalid inputs, error handling, and null values.
- **Use Mocks Effectively**: Mock dependencies (like network clients, databases) to isolate the unit under test. Verify interactions with mocks using `verify()`.
- **Integration Tests**: Write separate integration tests (potentially tagged with `@integration`) to verify the interaction between multiple components or with external systems. These often require more setup and may run slower.

## 4. Code Coverage

- **Tool**: `package:coverage`.
- **Generation**: Use the following commands to run tests and generate an LCOV report (suitable for tools like Codecov):
  ```bash
  # Ensure coverage package is activated globally or run via dart run
  dart pub global activate coverage

  # Run tests and collect coverage data
  dart test --coverage=coverage

  # Format coverage data into LCOV format
  dart pub global run coverage:format_coverage --lcov --in=coverage --out=coverage/lcov.info --packages=.dart_tool/package_config.json --report-on=lib
  ```
  *(Note: Adapt `--report-on` path if your code isn't solely in `lib`)*.
- **Goal**: Aim for high code coverage (e.g., >90-95%). Define specific thresholds per project/package.
- **CI Enforcement**:
    - Generate the LCOV report in CI.
    - Upload the report to a coverage service (e.g., Codecov, Coveralls) using their respective actions/tools.
    - Configure the coverage service to fail the build or PR check if coverage drops below the defined threshold or decreases significantly.
- **Coverage Gaps**: Regularly review coverage reports to identify untested code paths and add missing tests. Don't just chase numbers; ensure meaningful tests cover important logic.
- **Excluding Files**: Exclude generated files (`**/*.g.dart`, `**/*.freezed.dart`) and potentially boilerplate/configuration files from coverage reporting where testing adds little value. This can often be configured in the coverage service settings or by manipulating the LCOV file before upload.

## 5. Test Analytics (Optional)

- Services like Codecov offer Test Analytics. If using Codecov:
    - Generate a JUnit XML test report alongside coverage:
      ```bash
      dart test --reporter=junit --coverage=coverage > test-report.junit.xml
      ```
    - Upload the JUnit report to Codecov using their action (`codecov/test-results-action@v1`) in addition to the coverage report. This helps identify flaky or slow tests.