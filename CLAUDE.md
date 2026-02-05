# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Monix is a high-performance Scala library for asynchronous and reactive programming, targeting both JVM and Scala.js. It is a Typelevel project built on Cats and Cats-Effect 2.x.

## Build & Test Commands

Build tool: **sbt** (v1.12.0)

```bash
# Full CI validation (clean + compile + test + package for JVM & JS + mima + unidoc)
sbt ci-all

# JVM only: clean + compile + test + package + tracingTests
sbt ci-jvm

# Scala.js only: clean + compile + test + package
sbt ci-js

# Binary compatibility + documentation checks
sbt ci-meta

# Run all JVM tests for a specific submodule
sbt executionJVM/test
sbt evalJVM/test
sbt reactiveJVM/test
sbt catnapJVM/test
sbt tailJVM/test

# Run a single test suite (minitest framework)
sbt "evalJVM/testOnly monix.eval.TaskGatherSuite"

# Reactive Streams TCK compliance tests
sbt reactiveTests/test

# Stack tracing tests (cached + full modes)
sbt tracingTests/test

# Format check
sbt scalafmtCheckAll scalafmtSbtCheck

# Format code (also auto-runs on compile when not in CI)
sbt scalafmtAll scalafmtSbt
```

## Module Architecture

Dependency graph (top to bottom):

```
monix (root aggregator)
├── monix-reactive    → Observable (push-based reactive streams with back-pressure)
│   └── depends on: monix-execution, monix-eval
├── monix-tail        → Iterant (pull-based purely functional streaming)
│   └── depends on: monix-catnap
├── monix-eval        → Task, Coeval (side-effect suspension)
│   └── depends on: monix-execution, monix-catnap
├── monix-catnap      → Pure abstractions on Cats-Effect type classes
│   └── depends on: monix-execution
├── monix-execution   → Scheduler, Cancelable, CancelableFuture
│   ├── depends on: monix-execution-atomic, monix-internal-jctools (JVM)
│   └── monix-execution-atomic → Low-level atomic references
└── monix-java        → Java interop (JVM only, depends on execution + eval as provided)
```

## Cross-Compilation Structure

Each module follows this source layout:
```
monix-<module>/
├── shared/src/main/scala/    # Shared code (JVM + JS)
├── shared/src/test/scala/
├── jvm/src/main/scala/       # JVM-specific
├── js/src/main/scala/        # Scala.js-specific
└── (optional) atomic/        # Sub-module (e.g., in monix-execution)
```

Version-specific source directories are also supported: `scala-2`, `scala-2.13`, `scala-3`.

SBT project names use JVM/JS suffixes: `executionJVM`, `executionJS`, `evalJVM`, `evalJS`, etc. The root aggregators are `coreJVM` and `coreJS`.

## Scala Versions

- Scala 2.13.18 and 3.3.7 (extracted from `.github/workflows/build.yml` at build time)
- Compiler plugins (kind-projector, better-monadic-for) are disabled for Scala 3

## Code Conventions

- **Formatting**: Scalafmt 3.5.2, max 120 columns, `scala213source3` dialect (Scala 3 files use `scala3` dialect)
- **Testing framework**: Minitest — tests extend `SimpleTestSuite`
- **Internal code**: Lives in `monix.<module>.internal` packages (excluded from ScalaDoc)
- **License header**: Apache 2.0 required on all source files (enforced by sbt-header)
- **Binary compatibility**: MiMa checks against version 3.4.0 baseline
- **Test execution**: Tests run sequentially (`parallelExecution := false`)

## Key Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| Cats | 2.7.0 | Type classes |
| Cats-Effect | 2.5.5 | Effect type classes |
| JCTools | 3.3.0 | Concurrent queues (JVM, shaded) |
| Reactive Streams | 1.0.3 | Stream protocol (JVM) |
| Minitest | 2.9.6 | Test framework |

## JCTools Shading

JCTools is shaded into `monix.execution.internal.jctools.*` via sbt-assembly in the `monix-internal-jctools` subproject. This avoids classpath conflicts for downstream users.
