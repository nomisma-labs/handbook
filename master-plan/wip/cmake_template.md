# cmake_template: improvement review

Reviewed 11 November 2026.

**Repository license: Proprietary.** `cmake_template` remains private internal tooling under
the [licensing plan](../licensing.md).

This combines the starter-repository surveys into a short improvement backlog.
[memorandum.md](../../memorandum.md) sets intent;
[toolchain-layer.md](../toolchain-layer.md) defines the shared tooling responsibilities. Keep this
as a reference and backlog: the baseline records existing mechanisms, while the next-work list
identifies remaining coverage and tooling gaps. CI descriptions refer to workflow configuration,
not verified hosted runs.

See the [implementation reference](cmake_template_reference.md) for warning examples, module-design
reasoning, optional testing tools, and template synchronization mechanics.

## Existing baseline

- CMake helpers `nom_lib`, `nom_app`, Catch2-specific `nom_c2test`, and optional
  `nom_python_module`; submodule dependencies, package exports, and native named modules for
  Nomisma C++ library APIs. Libraries use `src/<component>/`.
- C++23 `import std;` is implemented for the supported toolchain. The
  [module setup](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/build.md#modules) defines the compatible
  compiler, standard-library and CMake versions. Include `cmake/kit/std_modules.cmake` before
  CXX detection and enable each consuming compiled target with `IMPORT_STD` in its kit helper.
  This is the template baseline, not the state of every research checkout; `trace_logger`
  still uses an older kit and the `core/` source layout.
- The native example has a primary interface, exported interface partitions, and implementation
  units. Apps, unit tests, and library usage consumers import `nom.example`. Headers remain for
  macros, including standard-library macros, and third-party integration boundaries.
- Catch2 unit tests in `tests/unit/` or beside the code; Catch2 discovery at the project root.
  Integration tests in `tests/integration/` and kit regression tests in `tests/kit/`, with
  separate `unit`, `integration`, and `kit` CTest labels.
- **Library usage integration tests** (`nom_library_usage_tests`): build and run a separate
  project using the library from source (`FetchContent`) and as an installed package
  (`find_package`). They check that another repo can import, link, and use the library.
- The [source-layout test](https://github.com/nomisma-labs/cmake_template/blob/master/tests/kit/source_layout/check.py)
  consumes a relocated installed module package after deleting its source and build trees.
  The [runtime relocation test](https://github.com/nomisma-labs/cmake_template/blob/master/tests/kit/relocation/check.py)
  runs an installed executable with transitive shared libraries after moving the installation
  and deleting the build tree.
- App `BUILD_FOR` selection and the `complete` preset; app configuration files collected during
  builds and installed with executables. Integration checks cover selection and config installation.
- `bootstrap.sh [preset]`: discovers presets through CMake, clears selected builds and old logs,
  then runs fresh builds/tests and quality checks with per-step results. Python compatibility/imports,
  SPDX license comments, formatting, and naming are checked; naming requires the `clang` preset.
- [CI](https://github.com/nomisma-labs/cmake_template/blob/master/.github/workflows/ci.yml) has a build matrix containing
  only `ci`, plus quality and release-version checks. `clang`, `complete`, and Clang ASan/UBSan
  builds remain part of default local bootstrap validation.
- The [template workflow](https://github.com/nomisma-labs/cmake_template/blob/master/.github/workflows/ci_template.yml)
  checks native Python wheels and that template-owned files stay generic. It does not run a
  renamed-repository initialization job.
- [Template synchronization](https://github.com/nomisma-labs/cmake_template/blob/master/tools/kit/sync_template.py) replaces
  template-owned files and stages `.template_ref` with them. Its
  [regression test](https://github.com/nomisma-labs/cmake_template/blob/master/tests/kit/template_sync/check.py) covers
  updates, moves, deletions, local-edit refusal, explicit overwrite, and repeat updates.

## Public examples worth using

| Source | Transferable practice | Application to Nomisma |
|---|---|---|
| cpp-best-practices: [warnings](https://github.com/cpp-best-practices/cmake_template/blob/b86318abbf55a657a11a0cf2acce69f221841b22/cmake/CompilerWarnings.cmake), [project options](https://github.com/cpp-best-practices/cmake_template/blob/b86318abbf55a657a11a0cf2acce69f221841b22/ProjectOptions.cmake), [Dependabot](https://github.com/cpp-best-practices/cmake_template/blob/b86318abbf55a657a11a0cf2acce69f221841b22/.github/dependabot.yml) | Compiler-specific warnings, tooling defaults that account for subproject use, automated Actions updates. | Extend the existing kit selectively; keep warnings and developer tools local to the project. |
| [Awesome C++](https://github.com/fffaraz/awesome-cpp) | Collection of C++ references. | Review individual tools when a concrete need appears. |
| ModernCppStarter: [install workflow](https://github.com/TheLartians/ModernCppStarter/blob/72b8957f9eb6971764d3433789b50d9653b7ec80/.github/workflows/install.yml) | Install the library, remove its build tree, then test it as a package. | Extend our relocated-package coverage to the complete example and its dependency graph. |
| Beman exemplar: [CMake configuration](https://github.com/bemanproject/exemplar/blob/32b69bd5846676013a204ce3c3fcbd6c074ea701/CMakeLists.txt), [generation checks](https://github.com/bemanproject/exemplar/blob/32b69bd5846676013a204ce3c3fcbd6c074ea701/cookiecutter/check_cookiecutter.sh) | Public-header verification and checks that generated projects contain the intended files and names. | Verify module interfaces and remaining public headers independently; add automated initialization-to-consumption coverage. |
| [beman-submodule](https://github.com/bemanproject/beman-submodule#how-does-it-work) | A committed infrastructure copy with an upstream revision and drift/update commands. | Use as a reference for reliable `sync_template.py` behavior; retain the local kit. |
| [friendlyanon/cmake-init](https://github.com/friendlyanon/cmake-init#goals) | Libraries usable by other projects without requiring their developer tools. | Test library usage as a dependency and scope optional checks to development builds. |

Use these as references for individual improvements within the existing template.

## Recommended next work

A **shared-dependency check** covers multiple dependency paths to one repository. The current
core facade supplies a concrete example: `nom_core → json_config → base` and
`nom_core → spd_logger → base`, alongside its direct `base` pin. This shape is called a
dependency *diamond*. All paths must use the selected `base` version.
A **local source override**, such as `FETCHCONTENT_SOURCE_DIR_BASE`, uses a developer's local
checkout before updating the committed dependency revision. The template documents this
mechanism; a dedicated diamond-and-override regression scenario remains useful.

| Priority | Improvement | Done when |
|---|---|---|
| 1 | **Test shared dependencies and local overrides.** | Add a dedicated fixture where the top-level project selects one shared revision, it is configured once, and a local checkout can replace it. Preserve the existing defaults that disable dependency tests/apps. Declare shared dependencies with consistent names before `FetchContent_MakeAvailable` ([ordering rules](https://cmake.org/cmake/help/latest/module/FetchContent.html)). |
| 1 | **Extend relocated-package coverage.** | The source-layout fixture already checks an importing consumer without the original source/build trees. Extend that pattern to the main example using `import std`, its third-party packages, and the complete exported dependency graph. |
| 1 | **Test the full initialization/update workflow.** | Initialize a renamed repo, add a small library/app, then build/test/install and consume its package after a template update. Preserve project files and keep removed examples out. The isolated sync regression already covers conflict refusal, overwrite, staged `.template_ref`, and repeat updates; the combined lifecycle remains to be automated. Follow the [sync instructions](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/new_repo.md#keeping-repos-in-sync). |
| 1, when needed | **Support private dependency checkout in CI.** | The template documents GitHub App tokens, but its workflows currently use ordinary recursive checkout. Add scoped read-only token wiring and verify access to all required private submodules before relying on hosted builds. |
| 2 | **Verify module interfaces and remaining public headers independently.** | Each public module builds and an importing consumer compiles. Separately compile each macro or dependency-boundary header to catch missing includes and accidental include-order requirements. `VERIFY_INTERFACE_HEADER_SETS` covers headers only; explicitly build `all_verify_interface_header_sets`, which is outside the default build ([CMake documentation](https://cmake.org/cmake/help/latest/prop_tgt/VERIFY_INTERFACE_HEADER_SETS.html)). |
| 2 | **Broaden compiler warnings.** | Add small compiler-specific groups, evaluating flags such as `-Wshadow`, `-Wconversion`, and `-Wformat=2`. Apply them to owned targets without imposing warnings-as-errors on projects using the library. |
| 2 | **Run sanitizers in CI.** | Run the existing `asan` preset, including native-module library usage integration tests, at an agreed cadence. |
| 2 | **Automate Actions updates.** | Add weekly reviewed Dependabot PRs for workflows and include its configuration in template synchronization. Assess submodule updates separately against release-pin policy and private-repo access. |

Independent module/header compilation is a proposed C++ correctness check. The existing
`tools/kit/checks/headers.py` checks SPDX license comments; it does not compile C++ headers or modules.
Bootstrap runs the existing checks; the remaining scenarios above need dedicated coverage.
Track module and toolchain experiments in the [lab checklist](https://github.com/nomisma-labs/cmake_template/blob/master/docs/lab/README.md).

## Add when there is a demonstrated need

- **Dev container:** reuse the CI image and presets when a second environment needs reproducible
  setup. Select a versioned image or digest and verify checkout, permissions, and tool versions;
  a moving `latest` tag alone does not make builds reproducible.
- **CMake formatting:** consider Beman's [gersemi/pre-commit setup](https://github.com/bemanproject/exemplar/blob/32b69bd5846676013a204ce3c3fcbd6c074ea701/.pre-commit-config.yaml).
  Keep a direct command usable by CI and agents; hooks can call the same command.
- **Coverage, assertions, TSan, fuzzing:** add separate options/presets for components with real
  logic, concurrent code, or parsers. Each addition needs a small example proving it works;
  release performance settings remain explicit.
- **Compiler caching:** check [ccache support](https://ccache.dev/manual/latest.html#_c_modules)
  for the chosen module/toolchain combination and measure representative builds before enabling it.
- **Additional platforms and toolchains:** validate both native modules and the implemented
  `import std` setup when testing compiler upgrades and macOS support. The current setup is
  tied to the supported CMake and standard-library module sources.
  [CMake's module documentation](https://cmake.org/cmake/help/latest/manual/cmake-cxxmodules.7.html)
  records supported combinations and limitations; it does not establish performance for Nomisma.

## Keep the template focused

Retain submodules plus FetchContent, the existing layout, presets, local initializer, and copied
CMake kit. New tooling must work through those choices and stay optional for projects using our
libraries. Public examples inform implementation; they do not require importing their package managers, example
libraries, platform matrix, or publishing workflows.

The discovery lists [Awesome Modern C++](https://github.com/rigtorp/awesome-modern-cpp) and
[awesome-cmake](https://github.com/onqtam/awesome-cmake) remain useful for finding examples.
