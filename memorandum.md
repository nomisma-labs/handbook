# Nomisma Labs — Statement of Intent

1. **Purpose.** Nomisma Labs is a development space for a new generation of trading platforms. The goal is to build a software workshop: reusable components, tools, and practices for constructing a family of trading systems that meet different business and technical requirements.

2. **Relation to other trading platforms.** The Nomisma platform is inspired by experience with an earlier crypto trading platform, Napoleon. Nomisma does not reuse its source code.

3. **System composition.** Each reusable component is developed in a separate repository, with its boundaries shaped by the needs of future systems. Each trading system will have a repository that brings together the components required for its use case. For example, an HFT system may require a dedicated performance testing framework, while a mid-frequency system may not. Separate component repositories support different combinations, access rights, licenses, and release cycles without keeping all projects in one monorepo.

4. **Development and distribution.** Components will have a private development version under a proprietary license, with the full development and agent tooling, skills, internal documentation, comprehensive tests, and CI/CD. Where distributed, a public or client version will carry an appropriate open-source or commercial license, with reduced documentation, test coverage, and CI/CD suited to its audience. Components intended solely for internal tooling, such as `cmake_template`, will remain private.

5. **Dependencies.** Repositories will depend on one another through Git submodules and explicit build-system dependencies. Shared components such as the configuration library, with [json_config](https://github.com/nomisma-labs/json_config) and [toml_config](https://github.com/nomisma-labs/toml_config) as JSON and TOML alternatives, and `logger` will support many repositories; each system repository will assemble a compatible set of component versions.

6. **Packaging and Distribution.** Nomisma Labs provides means for various types of packaging and distribution. Initially, we will support the following options: source-level distribution, pre-built libraries and binaries for a certain platform, Python "wheel" packages. Please see [packaging.md](master_plan/wip/packaging.md).

## Decisions

- **License:** private repositories start under a proprietary license, with `LICENSE` at the root. Copyright belongs to Alexey Tkachenko until transferred to Nomisma Labs. Distribution licenses are chosen per component. Public repositories will have an open source license (TBD).
- **SPDX:** every private Nomisma source file carries `SPDX-FileCopyrightText: 2026 Alexey Tkachenko` and `SPDX-License-Identifier: LicenseRef-Nomisma-Labs-Proprietary`.
- **Hosting:** GitHub organization, private repositories by default, GitHub Actions for CI/CD; start on the Free plan.
- **Language:** module-first C++, using the latest stable standard supported by the project toolchain. Use `nom` as the root namespace, component namespaces where useful, and standard-library-style `snake_case` naming.

## Architecture directory

| Layer | Responsibility | Components |
| --- | --- | --- |
| [Foundation layer](master_plan/foundation-layer.md) | Shared build infrastructure, utilities, composition, configuration and logging | `cmake_template`, `base`, `assembly`, configuration libraries and loggers. |
| [Service layer](master_plan/service-layer.md) | Reusable runtime services used by applications | `asio_comm` is the first completed component; multithreading facilities are future work. |
| [Application layer](master_plan/application-layer.md) | Operational workflows that compose foundation and service components | `md_capture` is the first planned component. |
