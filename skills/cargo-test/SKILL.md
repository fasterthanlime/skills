---
name: cargo-test
description: Use cargo nextest instead of cargo test for running Rust tests. This skill should be invoked whenever running tests in a Rust project. Cargo nextest provides superior test output, filtering with expression language, wrapper script integration, retries, and profiling capabilities. Always use `cargo nextest run` instead of `cargo test`.
---

# Cargo Nextest

## Overview

Cargo nextest is a next-generation test runner for Rust. **ALWAYS use `cargo nextest run` instead of `cargo test`** when working with Rust projects. Nextest provides better test output, powerful filtering, wrapper script integration, and many features not available in cargo test.

## Installation

If nextest is not installed, install it first:

```bash
cargo install cargo-nextest --locked
```

## Basic Usage

**CRITICAL:** Replace `cargo test` with `cargo nextest run`:

```bash
# ❌ WRONG
cargo test

# ✅ CORRECT
cargo nextest run
```

Run tests in a specific package:

```bash
cargo nextest run -p package_name
```

Run a specific test:

```bash
cargo nextest run test_name
```

Run tests matching a pattern:

```bash
cargo nextest run test_prefix
```

## Output and Capture Options

### --no-capture

Show test output in real-time (runs tests serially):

```bash
cargo nextest run --no-capture
```

This is the equivalent of `cargo test -- --nocapture`, but nextest handles it more cleanly.

### Status Levels

Control which test statuses are displayed:

```bash
# Show only failing tests during run
cargo nextest run --status-level fail

# Show all statuses (pass, fail, skip, slow, retry)
cargo nextest run --status-level all

# Control final summary output
cargo nextest run --final-status-level pass
```

Available status levels: `none`, `fail`, `retry`, `slow`, `pass`, `skip`, `all`

### Progress Display

Control progress bar and test counter:

```bash
# Hide progress bar
cargo nextest run --show-progress=none

# Show running tests only (up to 8)
cargo nextest run --show-progress=only
```

### Output Indentation

By default, nextest indents captured output with 4 spaces. Disable this:

```bash
cargo nextest run --no-output-indent
```

## Filtering with Expression Language (Filtersets)

Nextest supports a powerful domain-specific language for selecting tests called **filtersets**.

### Basic Filtering

Filter by test name pattern:

```bash
cargo nextest run -E 'test(test_name)'
```

Filter by package:

```bash
cargo nextest run -E 'package(mypackage)'
```

Filter by binary:

```bash
cargo nextest run -E 'binary(mybinary)'
```

### Complex Expressions

Combine filters with boolean operators:

```bash
# AND operator
cargo nextest run -E 'package(foo) and test(bar)'

# OR operator
cargo nextest run -E 'package(foo) or package(bar)'

# NOT operator
cargo nextest run -E 'not test(slow_test)'

# Grouping with parentheses (escape in shell!)
cargo nextest run -E 'package(foo) and (test(bar) or test(baz))'
```

### Common Filterset Predicates

- `test(name_pattern)` - Match test name
- `package(name_pattern)` - Match package name
- `binary(name_pattern)` - Match binary name
- `kind(test|bench|example)` - Match test kind
- `platform(cfg_expression)` - Match platform

**Note:** If using parentheses or brackets in shell, escape them or quote the entire expression.

## Configuration and Profiles

Nextest is configured via `.config/nextest.toml` in the project root.

### Profile Structure

Profiles define specific test configurations:

```toml
[profile.default]
retries = 0
fail-fast = true

[profile.ci]
retries = 2
fail-fast = false
test-threads = 4

[profile.ci.junit]
path = "junit.xml"
```

### Running with Profiles

```bash
cargo nextest run --profile ci
```

### Profile Options

Common profile settings:

- `retries` - Number of times to retry failed tests (default: 0)
- `fail-fast` - Stop on first failure (default: true)
- `test-threads` - Number of parallel test threads
- `run-wrapper` - Wrapper script to use (see below)
- `setup` - Setup scripts to run before tests
- `platform` - Platform-specific configuration using cfg expressions

## Wrapper Scripts

Wrapper scripts execute around test binaries for instrumentation, resource limits, or debugging.

### Configuring Wrappers

In `.config/nextest.toml`:

```toml
[scripts.wrapper.valgrind]
command = 'valgrind --leak-check=full --error-exitcode=1'

[scripts.wrapper.timeout]
command = 'timeout ${TIMEOUT:-60}'

[scripts.wrapper.limited]
command = 'systemd-run --user --scope -p MemoryMax=${MEMORY_LIMIT:-2G} -p MemorySwapMax=0'
platform = 'cfg(target_os = "linux")'

[profile.valgrind]
run-wrapper = 'valgrind'
test-threads = 1
platform = 'cfg(target_os = "linux")'
```

### Running with Wrappers

```bash
# Run tests with valgrind wrapper
cargo nextest run --profile valgrind

# Run tests with memory limits (Linux only)
cargo nextest run --profile limited
```

### Common Wrapper Use Cases

1. **Memory debugging** - valgrind, sanitizers
2. **Performance profiling** - callgrind, perf
3. **Resource limits** - systemd-run, timeout
4. **Custom instrumentation** - coverage tools, tracers

## Setup Scripts

Setup scripts run **before** tests start, useful for:

- Starting databases or services
- Creating test fixtures
- Preparing the environment

### Configuring Setup Scripts

```toml
[scripts.setup.start-db]
command = 'scripts/start-test-db.sh'

[profile.integration]
setup = ['start-db']
test-threads = 1
```

Run with setup:

```bash
cargo nextest run --profile integration
```

## Test Retries

Configure automatic retries for flaky tests:

### Command Line

```bash
# Retry failed tests up to 10 times
cargo nextest run --retries 10
```

### Configuration

```toml
[profile.default]
retries = 0

[profile.flaky]
retries = 3
retry-delay = { delay = "1s", backoff = "exponential" }
```

When a test succeeds after retrying, it's marked as **flaky**.

## JUnit XML Output

Enable JUnit XML output for CI systems:

```toml
[profile.ci.junit]
path = "junit.xml"
```

Output will be written to `target/nextest/ci/junit.xml`.

Run with:

```bash
cargo nextest run --profile ci
```

## Common Workflows

### Running All Tests

```bash
cargo nextest run
```

### Running Tests with Output

```bash
# Show all output
cargo nextest run --no-capture

# Show only failing test output (default behavior)
cargo nextest run
```

### Running Specific Tests

```bash
# Run one test
cargo nextest run specific_test_name

# Run tests in a package
cargo nextest run -p mypackage

# Run tests matching a pattern with expression language
cargo nextest run -E 'test(integration_)'
```

### Running Tests in CI

```bash
# Use CI profile with retries and no fail-fast
cargo nextest run --profile ci
```

### Debugging with Valgrind

```bash
# Run with valgrind wrapper (Linux only)
cargo nextest run --profile valgrind -p mypkg test_name
```

### Profiling with Callgrind

```bash
# Run with callgrind wrapper (requires configuration)
cargo nextest run --profile callgrind -p mypkg test_name
```

## Key Differences from cargo test

1. **Better output** - Cleaner, more informative test results
2. **Parallel by default** - Tests run in parallel unless --test-threads=1
3. **Filtersets** - Powerful expression language for test selection
4. **Wrapper scripts** - Built-in support for instrumentation and debugging
5. **Retries** - Automatic retry of flaky tests
6. **Setup scripts** - Run setup before tests
7. **Better CI integration** - JUnit XML, status levels, profiles
8. **--no-capture works better** - Cleaner handling of test output

## Platform-Specific Features

Some features are platform-specific and should use the `platform` key:

```toml
[profile.valgrind]
platform = 'cfg(target_os = "linux")'
run-wrapper = 'valgrind'

[profile.systemd-limits]
platform = 'cfg(target_os = "linux")'
run-wrapper = 'limited'
```

On unsupported platforms, nextest will skip the profile or show an error.

## Summary

**Always use cargo nextest instead of cargo test.** Key commands:

- `cargo nextest run` - Run all tests
- `cargo nextest run --no-capture` - Show output in real-time
- `cargo nextest run -E 'expression'` - Filter with expression language
- `cargo nextest run --profile name` - Use a configuration profile
- `cargo nextest run --retries N` - Retry failed tests
- `cargo nextest run --status-level all` - Show all test statuses

For more advanced usage, configure `.config/nextest.toml` with profiles, wrappers, and setup scripts.
