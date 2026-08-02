# test-brief-source

This is a test brief source for The Agency to use.

It holds the Claude files collected from the sibling repositories in `the-agency-hq`, laid out in the Brief source format `../the-agency` expects (`docs/design/2026-07-30-brief-pipeline-design.md` §8).

## Layout

```
the-agency-hq-settings.json     # layout marker, SemVer 1.0.0
rules/                          # shared -> .claude/rules/** AND .codex/rules/**
├── code-conventions.md         (+ .mission-types sibling)
├── copyright.md                (+ .mission-types sibling)
├── error-messages.md           (+ .mission-types sibling)
├── git.md                      (+ .mission-types sibling)
├── module-system.md            (+ .mission-types sibling)
└── worktrees.md                (+ .mission-types sibling)
skills/                         # shared -> .claude/skills/** AND .codex/skills/**
├── latte-build/
│   ├── .mission-types
│   └── SKILL.md
└── project-documentation/
    ├── .mission-types
    └── SKILL.md
claude/                         # verbatim escape hatch -> .claude/**
├── settings.json
└── settings.json.mission-types
```

`rules/` and `skills/` sit at the top level so every agent type gets them: the builder emits one Brief file per
agent type from a shared directory, so each rule and skill lands in both `.claude/**` and `.codex/**`. Adding a
third agent type is one entry in `OutputPaths.AGENT_TYPES` and needs no change here.

Only `settings.json` stays under the `claude/` escape hatch, because `.claude/settings.json` is a Claude-only file
with a Claude-only schema — there is no Codex equivalent for it to translate into.

`README.md` and `LICENSE` are root-level files outside the five mapped directories, so the builder ignores them.

## What happened to CLAUDE.md

`CLAUDE.md` was collected from `../handler` in the first pass and has been refactored away. Each section went to
the home that matches its shape — reference material an agent needs on demand became a skill, constraints that must
hold while editing particular files became a path-scoped rule:

| CLAUDE.md section | Became | Why |
|---|---|---|
| Documentation | skill `project-documentation` | The convention decides **where a new document goes**, so it has to fire on intent ("write a design doc"). A rule scoped to `docs/**` only fires once a file is already at a matching path, which is too late to prevent the mistake it exists to prevent. |
| Build & run | skill `latte-build` | A table of build commands is reference material wanted on demand, not context worth carrying always. |
| Module system | rule `module-system.md` | "Adding a dependency means editing `project.latte` **and** `module-info.java`" is a constraint, and its failure mode happens while editing exactly those two files. `paths:` scopes it to them, so it fires at the moment it is needed. |
| Worktree | rule `worktrees.md` | A standing convention, always applicable. |
| Project | dropped | It read "The Agency's Handler daemon" — a one-line identity statement for one repository. It is the only part of the file that could not generalize, and shipping it to a fleet would tell every Location it is the Handler. |

Three notes on the refactor:

1. **`worktrees.md` is its own rule rather than a section appended to `git.md`.** Topically it belongs in `git.md`,
   but `git.md` is byte-identical to the copy in all three source repositories, and this source is meant to be
   re-collected from them. Editing it turns every future re-collection into a merge. A separate file keeps the
   upstream files pristine and the locally-authored ones clearly ours.
2. **A stale reference was corrected.** CLAUDE.md's single-test example named
   `dev.theagencyhq.daemon.tests.UpdaterTest`. That package does not exist — `grep -r 'theagencyhq.daemon'` across
   `../handler` and `../the-agency` returns nothing; the real test module is `dev.theagencyhq.handler.tests`, and
   there is no `UpdaterTest`. The skill uses a placeholder FQCN instead, since a Brief goes to repositories whose
   test packages this source cannot know.
3. **The target table was widened to what all three repositories actually share.** CLAUDE.md listed six targets;
   `clean`, `build`, `test`, `int`, `release`, `idea`, `print-dependency-tree` and `run` are declared in
   `../handler`, `../foo` and `../the-agency` alike, so all eight are documented. Targets beyond that set are
   project-specific (`bundle` in the Handler; `codegen`, `tailwind` and the database targets in the Agency), and
   the skill says to read `project.latte` rather than listing them.

## Resulting Brief files

17 files — every shared file emits once per agent type:

| Brief path | Mission Types |
|---|---|
| `.claude/rules/code-conventions.md` · `.codex/rules/code-conventions.md` | `java` |
| `.claude/rules/copyright.md` · `.codex/rules/copyright.md` | `java` |
| `.claude/rules/error-messages.md` · `.codex/rules/error-messages.md` | `java` |
| `.claude/rules/git.md` · `.codex/rules/git.md` | `cli`, `java`, `library`, `web` |
| `.claude/rules/module-system.md` · `.codex/rules/module-system.md` | `java` |
| `.claude/rules/worktrees.md` · `.codex/rules/worktrees.md` | `cli`, `java`, `library`, `web` |
| `.claude/skills/latte-build/SKILL.md` · `.codex/skills/latte-build/SKILL.md` | `java` |
| `.claude/skills/project-documentation/SKILL.md` · `.codex/skills/project-documentation/SKILL.md` | `cli`, `java`, `library`, `web` |
| `.claude/settings.json` | `java` |

`BriefFile` canonicalizes Mission Types — trimmed, lowercased, deduplicated, sorted — because they feed the Brief's
checksum. The `.mission-types` files therefore keep the author's original case and order, while the table above
shows the canonical form that goes on the wire.

### Frontmatter rides along untranslated

The rule files carry Claude's YAML frontmatter (`paths:`), and the skills carry Claude's (`name:`, `description:`).
Both are emitted byte-for-byte into `.codex/**` as well, because milestone 1 maps paths and never rewrites bytes —
content translation between agent types is explicitly out of scope (design §2). This is expected, not a defect in
this source: it is the case that makes the deferred translation layer visible, which is useful in a test fixture.

## Mission Types

Both resolution mechanisms are used, matching what each shape of content needs:

- **`rules/` uses sibling `<file>.mission-types` files.** The directory is flat and its files are unrelated to one
  another, so types belong at each file.
- **`skills/` uses a directory-level `.mission-types` inside each skill.** A skill is one unit that may grow
  `scripts/`, `references/` and `assets/` later, and a directory file means those inherit automatically instead of
  each new file needing its own sibling. This is also how `idea.md` shows a skill laid out.

Every file in the Brief therefore resolves Mission Types, and the source exercises both branches of
`MissionTypeResolver` — which is worth having in something whose job is to be a test fixture.

The vocabulary is drawn from what the sibling repositories actually are:

- **`Java`** — `handler`, `foo`, `the-agency`. The `**/*.java` rules, the module-system rule and the `latte-build`
  skill are all Java/Latte-specific, as is `settings.json` (it allows `Bash(latte *)`).
- **`Web`** — `the-agency` (Latte web app) and `website` (Hugo).
- **`Library`** — reusable libraries.
- **`CLI`** — `handler` and `foo`, which are a daemon plus CLI.

`git.md`, `worktrees.md` and `project-documentation` list all four rather than being left with no types. Both mean
"everything" against the Locations that exist today, but an explicit list keeps the requirement that every file
carries a Mission Types file honest instead of satisfying it with an empty file.

## Sources

| Brief file | Origin |
|---|---|
| `rules/code-conventions.md` | collected — identical in all three repositories |
| `rules/error-messages.md` | collected — identical in all three repositories |
| `rules/git.md` | collected — identical in all three repositories |
| `claude/settings.json` | collected — identical in all three repositories |
| `rules/copyright.md` | collected — `../handler` (= `../foo`) |
| `rules/module-system.md` | authored here, from CLAUDE.md's "Module system" |
| `rules/worktrees.md` | authored here, from CLAUDE.md's "Worktree" |
| `skills/latte-build/` | authored here, from CLAUDE.md's "Build & run" |
| `skills/project-documentation/` | authored here, from CLAUDE.md's "Documentation" |

**`copyright.md` exists in two versions**, and a Brief path can only be produced once (§8.6 fails a build where two
sources collide on one output path). `../handler` and `../foo` carry an extra "Upstream files" section that
`../the-agency`'s copy lacks; the two are otherwise identical. The `../handler` version is used because it is the
superset, and its extra paragraph is conditional ("Some files in this project might be copied from upstream
projects"), so it is harmless where it does not apply.

`../internal-docs` and `../website` have no Claude files at all.
