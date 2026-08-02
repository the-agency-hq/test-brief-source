---
paths:
  - "**/module-info.java"
  - project.latte
---

# Module system

Both `src/main` and `src/test` are JPMS modules, each with its own `module-info.java`. The test module is separate
from the main module and `requires` it by name.

## Adding an external dependency takes two edits

A new dependency is not usable until it is declared in both places:

1. **`project.latte`** — add it to the appropriate `dependencies` group (`compile`, `test-compile`,
   `compile-processors`, and so on).
2. **`module-info.java`** — add a `requires` clause for its **module name** to whichever module uses it.

Doing only the first produces a compile error that names the *package* as not visible rather than the dependency as
missing, which reads like a typo in an import. Doing only the second fails to resolve the module at all. Neither
message points at the edit that was skipped, so make both edits together.

A dependency used only by tests goes in the test group and the test `module-info.java`. It does not belong in the
main module.

## Test access to internal packages

Tests live in their own module, so ordinary package-private and same-package access does not apply. A package the
tests need must be reachable from `src/main`'s `module-info.java`:

- **`exports <package>;`** — for a package the tests use normally.
- **`opens <package> to <consumer>;`** — for reflective access. jOOQ's generated record classes are one example;
  the test module `opens` its own test packages to `org.testng` so TestNG can reflect over the test classes.

Every test package containing test classes needs its own `opens ... to org.testng;` line. Adding a new test
sub-package and forgetting the `opens` produces a runner that silently finds no tests in it rather than an error.

## Ordering

`requires`, `exports`, and `opens` clauses are alphabetized, and the targets after `to` are alphabetized too. This
is the general alphabetization convention — see the `code-conventions` rule, which owns it.
