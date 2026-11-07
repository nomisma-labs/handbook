# cmake_template: improvement review

Updated 2026-10-04.

This combines the starter-repository surveys into a short improvement backlog.
[memorandum.md](../../memorandum.md) sets intent;
[toolchain-layer.md](../toolchain-layer.md) defines the shared tooling responsibilities. Proposed work below
is not yet implemented. CI descriptions refer to workflow configuration, not verified hosted runs.

See the [implementation reference](cmake_template_reference.md) for warning examples, module-design
reasoning, optional testing tools, and template synchronization mechanics.

## Existing baseline

- CMake helpers `nom_lib`, `nom_app`, and Catch2-specific `nom_c2test`; submodule dependencies,
  package exports, and native named modules for Nomisma C++ library APIs. The module-first
  template changes are under review.
- The native example has a primary interface, exported interface partitions, and implementation
  units. Apps, unit tests, and library usage consumers import `nom.example`. Headers remain for
  macros, standard-library declarations, and third-party integration boundaries.
- Catch2 unit tests in `tests/unit/` or beside the code; Catch2 discovery at the project root.
  Integration tests in `tests/integration/`, with separate CTest labels for both groups.
- **Library usage integration tests** (`nom_library_usage_tests`): build and run a separate
  project using the library from source (`FetchContent`) and as an installed package
  (`find_package`). They check that another repo can import, link, and use the library.
- App `BUILD_FOR` selection and the `complete` preset; app configuration files collected during
  builds and installed with executables. Integration checks cover selection and config installation.
- `bootstrap.sh [preset]`: discovers presets through CMake, clears selected builds and old logs,
  then runs fresh builds/tests and quality checks with per-step results. Python compatibility/imports,
  SPDX license comments, formatting, and naming are checked; naming requires the `clang` preset.
- [CI](../../../toolchain/cmake_template/.github/workflows/ci.yml) builds `ci`, `clang`, and `complete`.
  The Clang ASan/UBSan preset runs in default bootstrap validation but is not in that CI matrix.
  The [template workflow](../../../toolchain/cmake_template/.github/workflows/ci_template.yml) initializes and
  validates an empty renamed repo; [template synchronization](../../../toolchain/cmake_template/tools/kit/sync_template.py)
  also exists.

## Public examples worth using

| Source | Transferable practice | Application to Nomisma |
|---|---|---|
| cpp-best-practices: [warnings](https://github.com/cpp-best-practices/cmake_template/blob/b86318abbf55a657a11a0cf2acce69f221841b22/cmake/CompilerWarnings.cmake), [project options](https://github.com/cpp-best-practices/cmake_template/blob/b86318abbf55a657a11a0cf2acce69f221841b22/ProjectOptions.cmake), [Dependabot](https://github.com/cpp-best-practices/cmake_template/blob/b86318abbf55a657a11a0cf2acce69f221841b22/.github/dependabot.yml) | Compiler-specific warnings, tooling defaults that account for subproject use, automated Actions updates. | Extend the existing kit selectively; keep warnings and developer tools local to the project. |
| [Awsome C++ repo](https://github.com/fffaraz/awesome-cpp) | Big list of useful C++ references. | Review applicability and start using them |
| ModernCppStarter: [install workflow](https://github.com/TheLartians/ModernCppStarter/blob/72b8957f9eb6971764d3433789b50d9653b7ec80/.github/workflows/install.yml) | Install the library, remove its build tree, then test it as a package. | Strengthen our existing package tests to detect accidental build-tree dependencies. |
| Beman exemplar: [CMake configuration](https://github.com/bemanproject/exemplar/blob/32b69bd5846676013a204ce3c3fcbd6c074ea701/CMakeLists.txt), [generation checks](https://github.com/bemanproject/exemplar/blob/32b69bd5846676013a204ce3c3fcbd6c074ea701/cookiecutter/check_cookiecutter.sh) | Public-header verification and checks that generated projects contain the intended files and names. | Verify module interfaces and remaining public headers are self-contained; extend our initializer test beyond an empty project. |
| [beman-submodule](https://github.com/bemanproject/beman-submodule#how-does-it-work) | A committed infrastructure copy with an upstream revision and drift/update commands. | Use as a reference for reliable `sync_template.py` behavior; retain the local kit. |
| [friendlyanon/cmake-init](https://github.com/friendlyanon/cmake-init#goals) | Libraries usable by other projects without requiring their developer tools. | Test library usage as a dependency and scope optional checks to development builds. |

Use these as references for individual improvements within the existing template.

## Recommended next work

A **shared-dependency check** covers, for example, a trading app using both `orders` and `market_data`,
which each depend on `toml_config`: `app → orders → toml_config` and `app → market_data → toml_config`.
This shape is called a dependency *diamond*. Both paths must use one agreed version of `toml_config`.
A **local source override** means using a developer's local `toml_config` checkout to test changes
before updating the committed dependency revision.

| Priority | Improvement | Done when |
|---|---|---|
| 1 | **Test shared dependencies and local overrides.** | The top-level project selects one `toml_config` revision for both paths; it is configured once; a local checkout can replace it; dependency tests/apps stay off by default. Declare shared dependencies with consistent names before `FetchContent_MakeAvailable` ([ordering rules](https://cmake.org/cmake/help/latest/module/FetchContent.html)). |
| 1 | **Strengthen library usage integration tests.** | Extend the existing native-module source/package consumers and installed executable relocation check: move a library installation to a different directory, then build/run an importing consumer with the original source/build trees unavailable. Required dependencies and exported targets work through both source and package consumption. |
| 1 | **Test the full initialization/update workflow.** | Initialize a renamed repo, add a small library/app, then build/test/install. Apply a template update: preserve project files, report conflicts, keep removed template examples out, and record the matching `.template_ref`. Applying the same update again changes nothing. Follow the [sync instructions](../../../toolchain/cmake_template/docs/kit/new_repo.md#keeping-repos-in-sync), including staging `.template_ref`. |
| 1, when needed | **Support private dependency checkout.** | Before adding the first private dependency, provide GitHub App token support in template-owned workflows and verify recursive checkout with read-only access to the required repos. |
| 2 | **Verify module interfaces and remaining public headers independently.** | Each public module builds and an importing consumer compiles. Separately compile each macro or dependency-boundary header to catch missing includes and accidental include-order requirements. `VERIFY_INTERFACE_HEADER_SETS` covers headers only; explicitly build `all_verify_interface_header_sets`, which is outside the default build ([CMake documentation](https://cmake.org/cmake/help/latest/prop_tgt/VERIFY_INTERFACE_HEADER_SETS.html)). |
| 2 | **Broaden compiler warnings.** | Add small compiler-specific groups, evaluating flags such as `-Wshadow`, `-Wconversion`, and `-Wformat=2`. Apply them to owned targets without imposing warnings-as-errors on projects using the library. |
| 2 | **Run sanitizers in CI.** | Run the existing `asan` preset, including native-module library usage integration tests, at an agreed cadence. |
| 2 | **Automate Actions updates.** | Add weekly reviewed Dependabot PRs for workflows and include its configuration in template synchronization. Assess submodule updates separately against release-pin policy and private-repo access. |

Independent module/header compilation is a proposed C++ correctness check. The existing
`tools/check_headers.py` checks SPDX license comments; it does not compile C++ headers or modules.
Bootstrap runs the checks already implemented; the remaining scenarios above need dedicated tests.
Track module and toolchain experiments in the [lab checklist](../../../toolchain/cmake_template/docs/lab/README.md).

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
- **Additional platforms and toolchains:** validate the established native-module design when
  testing compiler upgrades and macOS support. Named modules use standard-library headers;
  investigate `import std` support separately.
  [CMake's module documentation](https://cmake.org/cmake/help/latest/manual/cmake-cxxmodules.7.html)
  records supported combinations and limitations; it does not establish performance for Nomisma.

## Keep the template focused

Retain submodules plus FetchContent, the existing layout, presets, local initializer, and copied
CMake kit. New tooling must work through those choices and stay optional for projects using our
libraries. Public examples inform implementation; they do not require importing their package managers, example
libraries, platform matrix, or publishing workflows.

The discovery lists [Awesome Modern C++](https://github.com/rigtorp/awesome-modern-cpp) and
[awesome-cmake](https://github.com/onqtam/awesome-cmake) remain useful for finding examples.
