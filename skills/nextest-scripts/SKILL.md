---
name: nextest-scripts
description: Configure cargo-nextest wrapper and setup scripts for test instrumentation. Use when integrating valgrind, memory limits, profiling, or custom test environments with nextest.
---

# Nextest Wrapper & Setup Scripts

Configure `cargo nextest` with wrapper scripts (valgrind, heaptrack, strace, perf) and setup scripts (build helpers, fixtures).

## Enabling Experimental Features

Wrapper and setup scripts require experimental flags in `.config/nextest.toml`:

```toml
# Enable both wrapper and setup scripts
experimental = ["wrapper-scripts", "setup-scripts"]

# Or just one:
# experimental = ["wrapper-scripts"]
# experimental = ["setup-scripts"]
```

## Wrapper Scripts

Wrapper scripts wrap test binary execution for instrumentation, debugging, or resource limits.

### Defining Wrappers

```toml
[scripts.wrapper.valgrind]
command = 'valgrind --leak-check=full --error-exitcode=1'

[scripts.wrapper.heaptrack]
command = 'heaptrack'

[scripts.wrapper.strace]
command = '/path/to/strace-wrapper.sh'

# Use relative-to for project scripts
[scripts.wrapper.custom]
command = { command-line = "scripts/wrapper.sh", relative-to = "workspace-root" }
```

### Applying Wrappers via Rules

Wrappers are applied using `[[profile.<name>.scripts]]` arrays:

```toml
[profile.valgrind]

[[profile.valgrind.scripts]]
filter = 'all()'
run-wrapper = 'valgrind'

# Platform-specific wrapper
[[profile.valgrind.scripts]]
platform = 'cfg(target_os = "linux")'
filter = 'all()'
run-wrapper = 'valgrind'
```

### list-wrapper vs run-wrapper

- `run-wrapper` - Wraps test execution
- `list-wrapper` - Wraps test listing (for cross-compilation/emulators like wine, qemu)

```toml
[[profile.windows-tests.scripts]]
filter = 'binary(windows_compat_tests)'
platform = { host = 'cfg(unix)', target = 'cfg(windows)' }
list-wrapper = 'wine-script'
run-wrapper = 'wine-script'
```

**Note:** If `list-wrapper` is used, `filter` cannot contain `test()` or `default()` predicates.

### Real-World Wrapper Examples

#### Valgrind (Memory Debugging)

```toml
[scripts.wrapper.valgrind]
# --leak-check=full: show details for each leak
# --show-leak-kinds=all: show all leak types for diagnostics
# --errors-for-leak-kinds=definite,indirect: only fail on real leaks
# --error-exitcode=1: exit with 1 if errors found
command = 'valgrind --leak-check=full --show-leak-kinds=all --errors-for-leak-kinds=definite,indirect --error-exitcode=1'

[scripts.wrapper.valgrind-lean]
# Faster than full leak detection, just catches runtime errors
command = 'valgrind --error-exitcode=1 --track-origins=yes --read-var-info=yes --verbose'

[profile.valgrind]
[[profile.valgrind.scripts]]
filter = 'all()'
run-wrapper = 'valgrind'

[profile.valgrind-lean]
[[profile.valgrind-lean.scripts]]
filter = 'all()'
run-wrapper = 'valgrind-lean'
```

#### Heaptrack (Memory Profiling)

```toml
[scripts.wrapper.heaptrack]
command = 'heaptrack'

[profile.heaptrack]
[[profile.heaptrack.scripts]]
platform = 'cfg(target_os = "linux")'
filter = 'all()'
run-wrapper = 'heaptrack'
```

#### Strace (Syscall Tracing)

```toml
[scripts.wrapper.strace]
command = { command-line = ".config/strace-wrapper.sh", relative-to = "workspace-root" }

[profile.strace]
[[profile.strace.scripts]]
platform = 'cfg(target_os = "linux")'
filter = 'all()'
run-wrapper = 'strace'
```

#### Perf (Performance Profiling)

```toml
[scripts.wrapper.perf]
command = { command-line = ".config/perf-wrapper.sh", relative-to = "workspace-root" }

[profile.perf]
[[profile.perf.scripts]]
platform = 'cfg(target_os = "linux")'
filter = 'all()'
run-wrapper = 'perf'
```

#### systemd-run (Memory Limits)

```toml
[scripts.wrapper.systemd-run]
command = 'systemd-run --user --scope -p MemoryMax=4G -p MemorySwapMax=0'

[profile.systemd-run]
[[profile.systemd-run.scripts]]
platform = 'cfg(target_os = "linux")'
filter = 'all()'
run-wrapper = 'systemd-run'
```

#### LLDB (Crash Debugging on macOS)

```toml
[scripts.wrapper.lldb]
command = { command-line = "scripts/lldb-wrapper.sh", relative-to = "workspace-root" }

[profile.lldb]
[[profile.lldb.scripts]]
platform = 'cfg(target_os = "macos")'
filter = 'all()'
run-wrapper = 'lldb'

[profile.debug]
# Allow 24 hours for manual debugging
slow-timeout = { period = "86400s" }
retries = 0
```

#### Sudo (Privileged Tests)

```toml
[scripts.wrapper.sudo-script]
command = 'sudo'

[[profile.ci.scripts]]
# IMPORTANT: Scope privileged tests as tightly as possible
filter = 'binary_id(package::binary) and test(=root_test)'
platform = 'cfg(target_os = "linux")'
run-wrapper = 'sudo-script'
```

**Warning:** Running tests as root can damage your system. Prefer running privileged tests in containers.

## Setup Scripts

Setup scripts run **before** tests for building helpers, starting services, or creating fixtures.

### Defining Setup Scripts

```toml
[scripts.setup.build-helpers]
command = "cargo build --bins -p helper-crate"
slow-timeout = { period = "60s" }

[scripts.setup.start-db]
command = { command-line = "scripts/start-test-db.sh", relative-to = "workspace-root" }
slow-timeout = { period = "60s", terminate-after = 2 }
leak-timeout = "1s"
capture-stdout = true
capture-stderr = false
```

### Applying Setup Scripts via Rules

```toml
# Run setup for all tests
[[profile.default.scripts]]
filter = 'all()'
setup = 'build-helpers'

# Platform-specific setup
[[profile.default.scripts]]
platform = { host = "cfg(unix)" }
filter = 'all()'
setup = 'set-env-unix'

[[profile.default.scripts]]
platform = { host = "cfg(windows)" }
filter = 'all()'
setup = 'set-env-windows'

# Multiple setup scripts
[[profile.integration.scripts]]
filter = 'rdeps(db-tests)'
setup = ['start-db', 'wait-db', 'seed-db']
```

### Environment Variables from Setup Scripts

Setup scripts can export environment variables to tests via `$NEXTEST_ENV`:

```bash
#!/usr/bin/env bash
# scripts/setup-test-env.sh

if [ -z "$NEXTEST_ENV" ]; then
    exit 1
fi

echo "DATABASE_URL=postgres://localhost:5432/test" >> "$NEXTEST_ENV"
echo "TEST_MODE=1" >> "$NEXTEST_ENV"
```

Tests matching the script's filter will see these environment variables.

### Real-World Setup Examples

#### Pre-Build Helper Binaries

```toml
[scripts.setup.build-helpers]
command = "cargo build --bins -p diagnostics -p http-handler -p template-engine"
slow-timeout = { period = "60s" }

[[profile.default.scripts]]
filter = 'all()'
setup = 'build-helpers'
```

#### Build Workspace Before Tests

```toml
[scripts.setup.build-cells]
command = 'cargo build --workspace --exclude devtools'

[[profile.default.scripts]]
platform = 'cfg(target_os = "linux")'
filter = 'not test(serve::)'
setup = 'build-cells'
```

## Complete Configuration Examples

### Minimal Setup-Only Config

```toml
experimental = ["setup-scripts"]

[scripts.setup.build-helpers]
command = "cargo build --bins -p helper-crate"
slow-timeout = { period = "60s" }

[profile.default]
fail-fast = true
slow-timeout = { period = "10s", terminate-after = 6 }
```

### Full Config with Wrappers and Setup

```toml
experimental = ["wrapper-scripts", "setup-scripts"]

# Setup scripts
[scripts.setup.set-env-unix]
command = 'scripts/setup-test-env.sh'

[scripts.setup.set-env-windows]
command = 'scripts/setup-test-env.cmd'

# Wrapper scripts
[scripts.wrapper.valgrind]
command = 'valgrind --leak-check=full --error-exitcode=1'

[scripts.wrapper.heaptrack]
command = 'heaptrack'

[scripts.wrapper.lldb]
command = { command-line = "scripts/lldb-wrapper.sh", relative-to = "workspace-root" }

# Default profile with platform-specific setup
[profile.default]
fail-fast = true

[[profile.default.scripts]]
filter = 'all()'
platform = { host = "cfg(unix)" }
setup = 'set-env-unix'

[[profile.default.scripts]]
filter = 'all()'
platform = { host = "cfg(windows)" }
setup = 'set-env-windows'

# Valgrind profile
[profile.valgrind]
[[profile.valgrind.scripts]]
filter = 'all()'
run-wrapper = 'valgrind'

# Heaptrack profile (Linux only)
[profile.heaptrack]
[[profile.heaptrack.scripts]]
platform = 'cfg(target_os = "linux")'
filter = 'all()'
run-wrapper = 'heaptrack'

# LLDB profile (macOS only)
[profile.lldb]
[[profile.lldb.scripts]]
platform = 'cfg(target_os = "macos")'
filter = 'all()'
run-wrapper = 'lldb'

# Debug profile for manual debugging
[profile.debug]
slow-timeout = { period = "86400s" }
retries = 0
```

### Config with Default Filter

```toml
experimental = ["wrapper-scripts", "setup-scripts"]

[profile.default]
test-threads = 4
# Exclude integration tests from default runs
default-filter = 'not test(serve::)'

[[profile.default.scripts]]
platform = 'cfg(target_os = "linux")'
filter = 'not test(serve::)'
setup = 'build-deps'
```

## Debugging

### Show Effective Configuration

```bash
cargo nextest show-config test
cargo nextest show-config test --profile valgrind
```

### Verbose Execution

```bash
cargo nextest run -v --profile valgrind test_name
```

### Test Wrapper Manually

```bash
valgrind ./target/debug/deps/mytest-abc123 test_name --nocapture
```

## Common Issues

### Wrapper Not Found

Ensure script is executable and use `relative-to`:

```toml
[scripts.wrapper.my-wrapper]
command = { command-line = "scripts/wrapper.sh", relative-to = "workspace-root" }
```

### Slow Tests Timeout

Increase timeout for instrumented tests:

```toml
[profile.valgrind]
slow-timeout = { period = "120s", terminate-after = 2 }
```

### Setup Script Timeout

Add `slow-timeout` to setup script definition:

```toml
[scripts.setup.build-helpers]
command = "cargo build --bins"
slow-timeout = { period = "120s" }
```

## See Also

- [nextest docs: Wrapper scripts](https://nexte.st/docs/configuration/wrapper-scripts/)
- [nextest docs: Setup scripts](https://nexte.st/docs/configuration/setup-scripts/)
- `valgrind` skill for memory debugging
- `systemd-run` skill for resource limits
