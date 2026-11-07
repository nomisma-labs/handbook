# Packaging and distribution

Updated 2026-10-04.

Status: work in progress. Native C++ source, libraries and applications are the core distribution
channels; Python packages provide an additional interface. The [memorandum](../../memorandum.md)
defines distribution intent and component independence; the
[toolchain layer](../toolchain-layer.md) defines shared build and delivery infrastructure.

## Scope and ownership

The toolchain layer, currently implemented by
[cmake_template](https://github.com/nomisma-labs/cmake_template), owns the build, dependency,
installation, packaging and validation mechanisms for every channel. Derived repositories
choose their targets, dependencies, package contents and audience through that framework.
Missing mechanisms are implemented and tested in the template before adoption elsewhere.

Component libraries remain independently usable from C++. System repositories assemble a
compatible selection of components. Each repository's version remains authoritative in CMake
`project()`; package metadata derives from it.

## Distribution channels

### Source

Consumers obtain a pinned Git revision through a submodule and incorporate it with CMake
`FetchContent`. System repositories select compatible dependency revisions. Consumers build
with the supported toolchain and component options. The template supports this path.

Source release archives still need a policy and validation for dependency contents, build
inputs and audience-specific contents.

### Native libraries and applications

Installed CMake packages provide compiled libraries, public headers where needed, module
interface sources and CMake metadata. Consumers use `find_package()` and exported targets.
The kit supplies installation/export rules and consumption checks.

C++ module BMIs are compiler-specific. Consumers rebuild them from the installed module
interfaces; the current checks use the same compiler as the library build. Rebuilding BMIs
does not make pre-built native libraries compatible across arbitrary compilers or C++ ABIs.

Application distributions contain executables, required packaged shared libraries and selected
configuration files. Existing install rules collect app configuration and provide relative
runtime-library paths. Tests must verify that the complete installed layout still works after
moving it and removing access to the original build. Required system libraries remain part of
the deployment contract.

Turning these installed layouts into release archives or OS packages, and publishing them,
requires additional shared release mechanisms. Installation support alone does not complete
that distribution workflow.

### Optional Python packages

A repository may expose selected source-built or imported C++ libraries through an importable
Python package. Wheels add a Python delivery format without adding Python packaging dependencies
to ordinary C++ builds. Packaging does not define the binding API.

The initial fixture is implemented locally: scikit-build-core drives CMake, the shared
helper installs an extension and declared runtime libraries, and tests exercise source-built
and imported libraries from installed wheels. Packaging tools use a separate pinned environment.
Repository initialization removes the example and retains reusable support. Operational
instructions live in the [template build guide](https://github.com/nomisma-labs/cmake_template/blob/master/docs/kit/build.md#native-python-wheels).

## Compatibility and audiences

Initial platform support follows the template's Ubuntu 26.04 baseline, with C++23, GCC 16 and
Clang 19 validation. The wheel fixture uses GCC 16 and CPython 3.14.4. Binary releases
identify tested CPU, OS/runtime and compiler compatibility; wheels also identify interpreter
compatibility. Broader portability requires validation.

Private, public and client distribution are audience choices across all three channels.
Components begin private; public and client releases follow the memorandum's licensing,
source-access and content rules. Artifact hosting, package indexes and publication credentials
are release concerns. Native releases do not require a Python index; public Python releases
may use PyPI. Publication workflows and client variants remain to be defined.

## Validation and remaining work

- **Source:** build a separate consumer from pinned sources and exercise the public API.
- **Native:** build a consumer against an installed CMake package; run an installed application
  with its configuration and packaged libraries after relocation, without the original build.
- **Python:** install the final wheel into a clean environment and call the C++ API, with original
  native artifacts unavailable; check metadata and runtime contents. Wheel installation requires
  no compiler, and ordinary C++ builds require no Python packaging tools.
- **Shared framework:** verify repository initialization and synchronization retain reusable
  mechanisms without restoring examples. Record the toolchain and dependency versions tested.

Next, review the template implementation and define native release contents and publication
mechanics. Release automation must validate the delivered archives or packages.
