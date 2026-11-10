# cmake_template: implementation reference

Reviewed 11 November 2026.

**Repository license: Proprietary.** `cmake_template` remains private internal tooling under
the [licensing plan](../licensing.md).

Supporting detail for the [improvement review](cmake_template.md). The review sets priorities;
this note preserves examples and reasoning. Proposed tooling options below are candidates to evaluate,
not features already enabled.

Source links use the [`src/` layout](../layers.md#repository-layout). Source moves in a derived
repository are separate from kit synchronization.

## Warning examples

The current [warning helper](https://github.com/nomisma-labs/cmake_template/blob/master/cmake/kit/internal/target_settings.cmake) enables
`-Wall -Wextra -Wpedantic` on owned compiled targets. Extend it in small groups, using the
[cpp-best-practices warning configuration](https://github.com/cpp-best-practices/cmake_template/blob/b86318abbf55a657a11a0cf2acce69f221841b22/cmake/CompilerWarnings.cmake)
as an implementation reference:

| Candidate flags | Purpose |
|---|---|
| `-Wshadow`, `-Wconversion` | Flag hidden variable names and potentially lossy implicit conversions. |
| `-Wformat=2`, `-Wimplicit-fallthrough` | Check printf-style formatting and unmarked switch fallthrough. |
| `-Wnon-virtual-dtor`, `-Woverloaded-virtual` | Expose mistakes in class hierarchies. |
| GCC-specific: `-Wduplicated-cond`, `-Wduplicated-branches`, `-Wlogical-op` | Expose suspicious repeated conditions, branches, and logical expressions. |

Check compiler support and the diagnostics on our examples before enabling each group.
Keep flags private to owned targets; warnings-as-errors remains a development/CI decision.

## Native modules and header boundaries

Nomisma C++ library APIs use native named modules. The template and the bundled core component
repositories implement this design; C++23 `import std;` support is also part of the current kit.
The `trace_logger` research checkout uses an older kit and has not adopted that toolchain.

The primary interface [example.cppm](https://github.com/nomisma-labs/cmake_template/blob/master/src/example/example.cppm)
declares `export module nom.example;` and re-exports the `counter`, `join`, `settings`, and `types`
interface partitions. Those partitions own the public declarations, including template
definitions needed by importers. Non-template definitions live in implementation units such as
[counter.cpp](https://github.com/nomisma-labs/cmake_template/blob/master/src/example/counter.cpp), which declares
`module nom.example;`. Apps, unit tests, and source/package consumers import `nom.example`.

Headers remain at specific boundaries:

- Macros are included from headers beside imports; they cannot be exported by a module.
- Each translation unit that uses standard-library declarations imports `std`. Standard macros
  still require their headers; macro-bearing and third-party includes precede imports.
- Third-party libraries can be wrapped behind a module API or used through headers where their
  integration requires it. Includes in a module's global module fragment precede its named
  module declaration.

Include `cmake/kit/std_modules.cmake` before the first `project()` enabling CXX, including in
source/package consumers. Use `IMPORT_STD` on consuming compiled targets created by the kit;
`nom_enable_std_modules(target)` serves targets created outside it. See the
[module setup](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/build.md#modules) for compatible
compiler, standard-library and CMake versions.

For example, [settings.cpp](https://github.com/nomisma-labs/cmake_template/blob/master/src/example/settings.cpp) includes
nlohmann privately in its global module fragment. The settings partition exposes parsing,
serialization, and loading through standard-library types; it does not export JSON types.
The dependency remains in installed package metadata because the static library's link interface
references its private target. Compiler-specific findings are recorded in
[module findings](https://github.com/nomisma-labs/cmake_template/blob/master/docs/lab/third_party_in_modules.md).

The [lab checklist](https://github.com/nomisma-labs/cmake_template/blob/master/docs/lab/README.md) records validation of native
interfaces, partitions, implementation units, importing tests, and source/package consumers on
GCC and Clang. Tooling, compiler upgrades, additional platforms, and measured build times remain
useful investigations within this design.

## Concrete testing options

Catch2 unit tests, library usage integration tests, and Clang ASan/UBSan already exist.
The following are optional additions for components that need them:

| Option | First useful application |
|---|---|
| libstdc++ assertions (`_GLIBCXX_ASSERTIONS`) | Check library preconditions in development builds. Inspect the toolchain defaults first; the macro enables checks such as string bounds and smart-pointer dereferences ([documentation](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_macros.html)). |
| Coverage with `llvm-cov` | Identify untested branches in configuration or order-processing logic using a dedicated instrumented build ([Clang coverage](https://clang.llvm.org/docs/SourceBasedCodeCoverage.html)). |
| ThreadSanitizer (`-fsanitize=thread`) | Run concurrent queue/runtime tests to detect data races, in a separate preset from ASan ([TSan](https://clang.llvm.org/docs/ThreadSanitizer.html)). |
| libFuzzer | Feed generated byte sequences into a small parser entry point; keep failure inputs as regression cases ([libFuzzer](https://llvm.org/docs/LibFuzzer.html)). |

Each addition should have a small example proving that it detects its intended problem.
Keep instrumentation explicit in development/test presets and assess release settings separately.

## How template synchronization works

Derived repos commit their own copy of the shared kit and tools. Their `.template_ref` records
the template URL and the exact upstream commit used as the update baseline.
The current [sync script](https://github.com/nomisma-labs/cmake_template/blob/master/tools/kit/sync_template.py):

1. Resolves the requested upstream commit/tag (`--to`), or the newest release tag by default,
   then runs that revision's copy of the sync script.
2. Requires a clean tracked worktree and no untracked files under template-owned paths.
3. Compares template-owned files with the recorded baseline and target. If local edits conflict,
   it lists them and changes nothing; explicit `--overwrite` selects the template's versions.
4. Replaces, adds or removes the selected files to match the target, without merging content
   or modifying repository-owned source and build declarations.
5. Records the new revision in `.template_ref` and stages it together with the synchronized files.
6. Reports stale references, repository-owned files inside kit folders, and intervening
   changelog upgrade steps that need manual attention.

Review the staged changes, including `.template_ref`, and validate before committing them
together. Commands are in [keeping repos in sync](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/new_repo.md#keeping-repos-in-sync).

Synchronization operates on committed template revisions and shared infrastructure paths; it
does not migrate a derived repo's framework code, tests, or project-specific CMake files.
Changes such as enabling `IMPORT_STD` or moving library sources to `src/` require coordinated
edits to those repository-owned files and compatible dependency pins.

[beman-submodule](https://github.com/bemanproject/beman-submodule#how-does-it-work) is a reference
for this model: committed infrastructure files, a recorded upstream revision, and explicit update
and status commands. A comparable status check could identify local deviations before updating.
For Nomisma, it must account for intentional initializer removals. The current
[sync regression](https://github.com/nomisma-labs/cmake_template/blob/master/tests/kit/template_sync/check.py) already checks
conflict refusal, explicit overwrite, preserved repository files, staged changes and repeat
updates. Combining initialization, real component builds and a later sync in one automated
scenario remains in the review backlog.
