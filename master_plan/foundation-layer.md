# Foundation layer: Build template, Integration Framework, Config Library, Logger

Updated 2026-10-13.

The [memorandum](../memorandum.md) records intent and platform decisions. This roadmap tracks
implementation order and remaining work; operational guidance lives in the component repos.

## 1. Sequence and current status

Each step should build, pass its tests, and provide the foundation needed by the next component.

| Repository | Role | Current work |
|---|---|---|
| [cmake_template](https://github.com/nomisma-labs/cmake_template) | Build template repo : shared CMake infrastructure, repository skeleton and feature lab | Module-first foundation released as `v0.8.2`; further improvements arrive as derived repos need them. |
| [assembly](https://github.com/nomisma-labs/assembly) | Integration framework: Node/context composition, bidirectional relay and callable delegates | Framework complete; released as `v0.2.0`. |
| [json_config](https://github.com/nomisma-labs/json_config), [toml_config](https://github.com/nomisma-labs/toml_config) | Application configuration library with JSON and TOML implementations | Shared design and similar capabilities for layered configuration, tools and C++ integration |
| [trace_logger](https://github.com/nomisma-labs/trace_logger) | High performance logging services and macro interface | Created, early stage |

Each trading system has a container repository that selects compatible component versions.
The [template backlog](wip/cmake_template.md) lists proposed improvements; they are introduced
as needed rather than making every lab experiment a prerequisite for the next repository.

## 2. Conventions

Use `nom` as the root namespace, `nom_` for shared prefixes, and standard-library-style
`snake_case`. Template parameters use `PascalCase`; macros use UPPER_CASE. Module names and
namespace hierarchies are independent: `nom.assembly` exposes its API directly in `nom`.

Nomisma declarations live in native module interfaces and partitions; compiled definitions use
implementation units. Apps and tests import the library's primary module. Macros stay in headers;
third-party dependencies use a tested module wrapper or header boundary. Each file includes the
standard-library headers it uses; `import std;` remains a toolchain investigation.

The canonical [conventions](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/conventions.md) define naming, file layout,
target names and enforcement. The [module rules](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/build.md#modules)
cover composition and dependency boundaries. Every source file carries the SPDX comments from
the memorandum.

## 3. Build and dependency rules

- Use the shared CMake kit and target-based dependencies. Tests and apps default to enabled
  only for a top-level project, so consuming a library does not build its development tools.
- Pin our libraries as Git submodules and integrate them through FetchContent. The system repo
  selects shared dependency versions and declares their pins before populating dependencies;
  consistent names let multiple dependency paths resolve to one instance.
- Pin third-party source dependencies to releases and integrate them case by case. Use
  `find_package` for system libraries. Local source overrides support co-development before
  updating a submodule pin.
- Support both source consumption and installed packages. Install module interface sources
  and required headers; consumers build their own compiler-specific BMIs.
- Keep shared infrastructure changes in `cmake_template`, then synchronize derived repos.
  Each repo records its template revision in `.template_ref`.

The [build guide](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/build.md) covers presets, dependency integration,
installation and tooling. Use `./bootstrap.sh` for fresh validation and preset build/test commands
for incremental work.

Every repo starts versioned at `0.1.0`, follows Semantic Versioning, and keeps its version in
`project()`. Annotated `vX.Y.Z` tags identify releases on `master`; CI checks tag/version agreement.
The [release procedure](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/build.md#versioning) is maintained in the template.

## 4. Build Template

The template supplies the shared kit, presets, checks, CI, agent guidance and recipes. It also
serves as the lab for features that will affect other repositories. Its native `nom.example`
module demonstrates partitions, template definitions, implementation units, a macro header and
private JSON integration. Importing tests cover source and installed-package consumption;
other integration checks cover app selection and installation relocation.

Create repositories with the [new-repo procedure](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/new_repo.md).
The initializer removes the example and lab, leaving an empty repository that grows through
[code recipes](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/howto.md). Repository settings and private dependency
access need their own setup.

For updates, follow the [sync procedure](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/new_repo.md#keeping-repos-in-sync).
It transfers shared infrastructure from committed template revisions. A composition change also
requires migrating each repo's own code, tests and CMake declarations; coordinate those changes
with the sync and validate source and package consumption together.

Releases run from `v0.1.0` to `v0.8.2`; the native-module change arrived in `v0.3.0`. The
[changelog](https://github.com/nomisma-labs/cmake_template/blob/master/CHANGELOG.md) records each
release and the upgrade steps derived repos take by hand. Current local validation and remaining
experiments are recorded in the [lab](https://github.com/nomisma-labs/cmake_template/blob/master/docs/lab/README.md); the [backlog](wip/cmake_template.md) tracks
infrastructure work beyond the implemented baseline.

## 5. Assembly : Integration Framework

Assembly composes processing nodes at compile time. Relay passes requests forward and responses
back through matching handlers; context mixins contribute shared state and services. Delegates
connect runtime event sources to the static framework. Assembly custom traits configure node
policies and list the events without a final consumer, which the relay suppresses at the end of
each route. An exception escaping event processing is fatal by policy. Metal supplies the
typelist machinery inside the module; applications declare lists with `nom::type_list`.

Start with **[How to understand Assembly/Relay applications](https://github.com/nomisma-labs/assembly/blob/master/docs/assembly/understanding_applications.md)**:
read the node list, then identify context services, event handlers, trait choices, configuration
reads, and original event emitters.

Assembly is the first library created from the template and exercises template-heavy module
code. Review the [framework guide](https://github.com/nomisma-labs/assembly/blob/master/docs/assembly/framework.md) and
[teaching tests](https://github.com/nomisma-labs/assembly/blob/master/docs/assembly/tests.md).

The framework is complete and released as `v0.2.0`, synced with cmake_template `v0.8.2`. Its five
teaching suites (`assembly`, `custom_traits`, `relay`, `complex_relay` and `delegate`) and
source/package consumers pass locally on GCC and Clang.

## 6. App Config

The application configuration library has two implementations:
[json_config](https://github.com/nomisma-labs/json_config) based on JSON and
[toml_config](https://github.com/nomisma-labs/toml_config) based on TOML. They share the same
design and are intended to provide similar capabilities.

Planned capabilities cover build-system configuration collection, a Python tool (`cfggen.py`)
that assembles runtime configuration from shared pieces and layers, C++ configuration discovery,
and mapping configuration values onto C++ types. Each implementation provides standalone tools
and C++ library support. Configuration macros stay in headers; reusable types and services use
modules. Validate each parser dependency boundary for that integration.

Apps and components choose the configuration implementation they need; Assembly and logger
remain independent of either implementation.
This step also exercises a tool run at build time, configuration installation, and CI access
to a private component dependency.

## 7. Logger

Logger provides levels, logging macros and runtime-configured file loggers. Its types and
services use C++20 modules; macros stay in headers.

## 8. Hosting, CI and documentation

The organization is `nomisma-labs`; repositories use `master` as the default branch. Privacy,
licenses and distribution policy are recorded in the memorandum.

The shared [CI workflow](https://github.com/nomisma-labs/cmake_template/blob/master/.github/workflows/ci.yml) currently builds the
`ci` preset and runs Python, SPDX, formatting, naming and release-tag checks. Broader compiler,
sanitizer and initializer coverage is tracked in the template backlog. Hosted run status and
image publication need verification separately from local validation.

The CI tool image should be public; it contains public tools. Before adding the first private
component dependency, add a read-only GitHub App checkout token to the template workflows and
sync that support into derived repos. Setup instructions live in the new-repo guide.

Cross-repo decisions and plans live in this handbook. Component APIs, recipes, validation results
and compiler findings live with their code. Agents use each repo's `AGENTS.md`, the shared
conventions and recipes, and update relevant documentation alongside code changes.

## 9. Toolchain and experiments

C++23 is the current validated baseline. Presets already use GCC 16 and Clang 19; development and
CI target Ubuntu 26.04 with Ninja. Exact versions, including GCC's development-snapshot status,
are recorded in the [toolchain guide](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/build.md#toolchain) and
[CI image](https://github.com/nomisma-labs/cmake_template/blob/master/docker/ci/Dockerfile).

Validate compiler or standard upgrades in the lab before changing the shared baseline.
The [lab checklist](https://github.com/nomisma-labs/cmake_template/blob/master/docs/lab/README.md) owns remaining module performance,
IDE/debugger, platform and standard-library investigations, along with historical findings.
