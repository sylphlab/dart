# Dart Performance Benchmarking

Measuring and tracking performance is important for critical code paths.

## 1. Benchmarking Tool

- **Standard Tool**: `package:benchmark_harness`. Provides a simple foundation for writing and running benchmarks.

## 2. Writing Benchmarks

- **Structure**: Create benchmark classes extending `BenchmarkBase`.
- **`run()` Method**: Place the code to be measured inside the `run()` method. This method will be executed multiple times to get a stable measurement.
- **`setup()` / `teardown()`**: Use these methods for any setup or cleanup needed before/after the `run()` loop, but *not* for work that should be included in the measurement itself.
- **Emitter**: Define a `ScoreEmitter` (or use the default `PrintEmitter`) to report results.

```dart
import 'package:benchmark_harness/benchmark_harness.dart';

// Example: Benchmark a string concatenation method

String concatenateStrings(List<String> strings) {
  final buffer = StringBuffer();
  for (final s in strings) {
    buffer.write(s);
  }
  return buffer.toString();
}

class StringConcatenationEmitter implements ScoreEmitter {
  final List<double> runtimes = [];

  @override
  void emit(String testName, double value) {
    // value is microseconds per run() call
    print('$testName(RunTime): $value us.');
    runtimes.add(value);
  }
}

class StringConcatenationBenchmark extends BenchmarkBase {
  final List<String> data;
  final StringConcatenationEmitter _emitter;

  StringConcatenationBenchmark(int size)
      : data = List.generate(size, (i) => 'string $i'),
        _emitter = StringConcatenationEmitter(),
        super('StringConcatenation(size: $size)', emitter: _emitter);

  @override
  void run() {
    concatenateStrings(data); // The code being measured
  }

  @override
  void setup() {
    // Optional setup before the run() loop starts
  }

  @override
  void exercise() {
     // Run the benchmark multiple times to warm up and get stable results
     for (int i = 0; i < 10; i++) { // benchmark_harness runs exercise() 10 times by default
       run();
     }
  }

  @override
  void teardown() {
    // Optional cleanup after the run() loop finishes
  }

  // Optional: Calculate average runtime after benchmark completes
  double get averageRuntime => _emitter.runtimes.isEmpty
      ? 0.0
      : _emitter.runtimes.reduce((a, b) => a + b) / _emitter.runtimes.length;
}

void main() {
  final benchmarkSmall = StringConcatenationBenchmark(100);
  benchmarkSmall.report();
  print('Average runtime (small): ${benchmarkSmall.averageRuntime} us.');

  final benchmarkLarge = StringConcatenationBenchmark(10000);
  benchmarkLarge.report();
  print('Average runtime (large): ${benchmarkLarge.averageRuntime} us.');
}

```

## 3. Running Benchmarks

- Place benchmark files in the `benchmark/` directory.
- Run them using `dart run benchmark/my_benchmark.dart`.

## 4. Performance Metrics & Goals

- **Primary Metric**: Time per operation (microseconds, milliseconds). `benchmark_harness` reports this.
- **Memory Usage**: Use Dart DevTools (`dart --observe run benchmark/my_benchmark.dart`) to profile memory allocation during benchmark runs if memory is a concern.
- **Comparisons**: Benchmark against previous versions or alternative implementations to measure improvements or regressions.
- **Performance Budgets**: For critical applications, define performance budgets (e.g., "login must complete under 500ms") and potentially integrate checks into CI, although this can be complex due to environment variations.

## 5. CI Integration (Optional)

- Running benchmarks in CI can be valuable but tricky due to potential variations in runner performance.
- **Approach**:
    - Run benchmarks on a dedicated, consistent runner if possible.
    - Store results (e.g., JSON output) as artifacts.
    - Compare results against a baseline (e.g., results from the `main` branch).
    - Report significant regressions as warnings or failures.
    - Consider tools or frameworks designed for CI benchmarking if high precision is needed.