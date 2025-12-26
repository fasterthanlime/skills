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

**Note:** Wrapper scripts are experimental. Enable with:
```toml
experimental = ["wrapper-scripts"]
```

### Defining Wrapper Scripts

In `.config/nextest.toml`:

```toml
experimental = ["wrapper-scripts"]

[scripts.wrapper.valgrind]
command = 'valgrind --leak-check=full --error-exitcode=1'

[scripts.wrapper.sudo-script]
command = 'sudo'

[scripts.wrapper.wine-script]
command = 'wine'

# Use relative-to for scripts in your project
[scripts.wrapper.project-wrapper]
command = { command-line = "scripts/wrapper.sh", relative-to = "workspace-root" }
```

### Configuring Wrapper Rules

Wrappers are applied via `[[profile.<name>.scripts]]` rules with filtersets:

```toml
# Run specific tests with sudo (Linux only)
[[profile.ci.scripts]]
filter = 'binary_id(package::binary) and test(=root_test)'
platform = 'cfg(target_os = "linux")'
run-wrapper = 'sudo-script'

# Use wine for Windows tests on Unix hosts
[[profile.windows-tests.scripts]]
filter = 'binary(windows_compat_tests)'
platform = { host = 'cfg(unix)', target = 'cfg(windows)' }
list-wrapper = 'wine-script'
run-wrapper = 'wine-script'
```

### list-wrapper vs run-wrapper

- `run-wrapper` - Used when executing tests
- `list-wrapper` - Used when listing tests (for cross-compilation/emulators)

**Note:** If `list-wrapper` is specified, `filter` cannot contain `test()` or `default()` predicates.

### Running with Wrappers

```bash
# Run tests with ci profile (applies wrapper rules)
cargo nextest run --profile ci

# Run tests for Windows on Unix host
cargo nextest run --profile windows-tests
```

### Common Wrapper Use Cases

1. **Memory debugging** - valgrind, sanitizers
2. **Performance profiling** - callgrind, perf
3. **Resource limits** - systemd-run, timeout
4. **Privilege escalation** - sudo (for tests requiring root)
5. **Cross-compilation** - wine, qemu

## Setup Scripts

Setup scripts run **before** tests start, useful for:

- Starting databases or services
- Creating test fixtures
- Preparing the environment

**Note:** Setup scripts are experimental. Enable with:
```toml
experimental = ["setup-scripts"]
```

### Defining Setup Scripts

```toml
experimental = ["setup-scripts"]

[scripts.setup.db-generate]
command = 'cargo run -p db-generate'
slow-timeout = { period = "60s", terminate-after = 2 }
leak-timeout = "1s"
capture-stdout = true
capture-stderr = false

[scripts.setup.start-db]
command = { command-line = "scripts/start-test-db.sh", relative-to = "workspace-root" }
```

### Configuring Setup Script Rules

Setup scripts are applied via `[[profile.<name>.scripts]]` rules:

```toml
# Run db-generate for tests that depend on db-tests package
[[profile.default.scripts]]
filter = 'rdeps(db-tests)'
setup = 'db-generate'

# Platform-specific setup
[[profile.default.scripts]]
platform = { host = "cfg(unix)" }
setup = 'unix-setup'

# Multiple setup scripts for integration tests
[[profile.integration.scripts]]
filter = 'test(/^script_tests::/)'
setup = ['start-db', 'seed-db']
```

### Environment Variables from Setup Scripts

Setup scripts can export environment variables to tests via `$NEXTEST_ENV`:

```bash
#!/usr/bin/env bash
# my-env-script.sh
if [ -z "$NEXTEST_ENV" ]; then
    exit 1
fi
echo "MY_ENV_VAR=Hello, world!" >> "$NEXTEST_ENV"
```

Tests matching the script's filter will see `MY_ENV_VAR`.

### Running with Setup

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

Platform filtering works both at the profile level and in script rules:

```toml
# Profile-level platform restriction
[profile.valgrind]
platform = 'cfg(target_os = "linux")'

# Script rule with platform filter
[[profile.ci.scripts]]
filter = 'binary_id(pkg::bin) and test(=root_test)'
platform = 'cfg(target_os = "linux")'
run-wrapper = 'sudo-script'

# Cross-compilation: host vs target platform
[[profile.cross.scripts]]
platform = { host = 'cfg(unix)', target = 'cfg(windows)' }
list-wrapper = 'wine-script'
run-wrapper = 'wine-script'
```

On unsupported platforms, nextest will skip the profile or rule.

## Summary

**Always use cargo nextest instead of cargo test.** Key commands:

- `cargo nextest run` - Run all tests
- `cargo nextest run --no-capture` - Show output in real-time
- `cargo nextest run -E 'expression'` - Filter with expression language
- `cargo nextest run --profile name` - Use a configuration profile
- `cargo nextest run --retries N` - Retry failed tests
- `cargo nextest run --status-level all` - Show all test statuses

For more advanced usage, configure `.config/nextest.toml` with profiles, wrappers, and setup scripts.
