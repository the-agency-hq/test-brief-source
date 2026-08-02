---
name: latte-build
description: Use when building, compiling, testing, running, or releasing a project whose root contains a `project.latte` — including running a single test, refreshing the IntelliJ module, or adding an external dependency. These projects use the Latte build tool, not Maven or Gradle.
---

# Latte build targets

This project is built with **Latte**. The build file is `project.latte` at the repository root, and Java 25 is
required — the version is set in `project.latte` under `java.settings.javaVersion`.

There is no `pom.xml` and no `build.gradle`. Do not create one, and do not reach for `mvn` or `gradle` when a build
command fails — check `project.latte` instead.

## Targets

These targets exist in every Latte project here:

| Task                        | Command                                    |
|-----------------------------|--------------------------------------------|
| Compile + JAR               | `latte build`                              |
| Run tests                   | `latte test` (depends on `build`)          |
| Run a single test           | `latte test --test=<fully.qualified.Class>`|
| Run the application         | `latte run` (depends on `build`)           |
| Local integration release   | `latte int` (depends on `test`)            |
| Full release                | `latte release` (depends on `clean`, `test`)|
| Refresh the IntelliJ module | `latte idea`                               |
| Print the dependency tree   | `latte print-dependency-tree`              |
| Clean                       | `latte clean`                              |

`latte int` publishes to the local integration repository, so a downstream project can depend on the build without
a real release.

## Project-specific targets

The table above is the common set, not the whole set. Individual projects add their own — database creation, code
generation, asset pipelines, native bundling. **Read `project.latte` before assuming a target does or does not
exist**: every `target(...)` declares a `description:`, so the file is its own reference.

## Running one test

`--test=` takes the **fully-qualified class name**, not a file path and not a bare class name:

```bash
latte test --test=com.example.project.tests.WidgetTest
```

Read the test's `package` declaration to build the argument rather than inferring it from the directory — the
package and the source path agree by convention, but the package is what the runner resolves.

## Build output

Build products land under `build/`, which is git-ignored:

| Path                 | Holds                                    |
|----------------------|------------------------------------------|
| `build/classes/`     | Compiled classes, `main` and `test`      |
| `build/jars/`        | The JARs produced by `latte build`       |
| `build/test-reports/`| TestNG HTML and JUnit XML reports        |

When a test fails and the console output is truncated, the full report is in `build/test-reports/`.

## Adding a dependency

Adding an external dependency requires editing **both** `project.latte` and a `module-info.java`. See the
`module-system` rule, which is scoped to exactly those two files.
