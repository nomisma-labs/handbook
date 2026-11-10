# Nomisma Labs — Statement of Intent

1. **Purpose.** Nomisma Labs is a development space for a new generation of trading platforms. The goal is to build a software workshop: reusable components, tools, and practices for constructing a family of trading systems that meet different business and technical requirements.

2. **Relation to other trading platforms.** The Nomisma platform is inspired by experience with an earlier crypto trading platform, Napoleon. Nomisma does not reuse its source code.

3. **System composition.** Each reusable component is developed in a separate repository, with its boundaries shaped by the needs of future systems. Each trading system will have a repository that brings together the components required for its use case. For example, an HFT system may require a dedicated performance testing framework, while a mid-frequency system may not. Separate component repositories support different combinations, access rights, licenses, and release cycles without keeping all projects in one monorepo.

4. **Development and visibility.** Work is organized in private repositories using shared conventions, developer and agent tooling, automated tests and CI/CD. Each repository maintains its implementation and technical documentation; cross-repository plans and decisions live in this handbook. A repository may become public once it reaches maturity, at its author’s discretion.

5. **Dependencies.** Repositories depend on one another through Git submodules and explicit build-system dependencies. Components reuse common facilities, such as configuration and logging, through these dependencies. The [layer plans](master-plan/layers.md) describe the dependency structure and ownership boundaries.

6. **Packaging and distribution.** See the [dedicated plan](master-plan/wip/packaging.md) for delivery formats and distribution workflows.

## Decisions

- **Copyright:** copyright belongs to Alexey Tkachenko.
- **License:** MIT for general low-level components, including the core libraries; Sustainable Use License 1.0 for repositories containing more domain knowledge; proprietary for distinctive know-how, including `trace_logger`, and private internal tooling. `trace_logger` is an explicit exception to the core layer's MIT default. Each distributed repository records its terms in `LICENSE`; see [licensing.md](master-plan/licensing.md).
- **SPDX:** copyright notices identify Alexey Tkachenko. Source license identifiers must match the actual license of the repository or distribution. Existing proprietary development sources use `SPDX-FileCopyrightText: 2026 Alexey Tkachenko` and `SPDX-License-Identifier: LicenseRef-Nomisma-Labs-Proprietary`; MIT distributions use `SPDX-License-Identifier: MIT`. SUL distributions must identify and include the selected Sustainable Use License 1.0 text.
- **Hosting:** GitHub organization, private repositories by default, GitHub Actions for CI/CD; start on the Free plan.
- **Language:** module-first C++, using the standard supported by the validated template toolchain. Adopt newer standards after validating the required compiler and library support. Use `nom` as the root namespace, component namespaces where useful, and standard-library-style `snake_case` naming.

## Architecture directory

See [Layers and integration boundaries](master-plan/layers.md) for dependency direction and
ownership of shared configuration adapters and application services.

The handbook records component roles, architectural boundaries, licensing decisions and planned
work. Exact dependency pins, toolchain versions, release histories and detailed validation results
belong in the component repositories. Link to those sources instead of duplicating their current
versions here.

| Layer | Responsibility | Components |
| --- | --- | --- |
| [Toolchain layer](master-plan/toolchain-layer.md) | Development, build, validation and delivery infrastructure for all code layers | `cmake_template` (**Proprietary**); the layer name is independent of its implementation. |
| [Core layer](master-plan/core-layer.md) | Common definitions, composition, configuration, logging, adapters and planned performance measurement | `nom_core` integrates `base`, `assembly`, `json_config` and `spd_logger` (**all MIT**); `toml_config` and `yaml_config` (**MIT**), `trace_logger` (**Proprietary**) and planned performance measurement (**MIT**) remain outside the facade. |
| [Service layer](master-plan/service-layer.md) | Reusable runtime services used by applications | `asio_comm` (**SUL 1.0**) is implemented, with further validation and feature work outstanding; `nom_services` (**SUL 1.0**) and multithreading facilities are planned. |
| [Application layer](master-plan/application-layer.md) | Operational workflows composed from services | `md_capture` (**SUL 1.0**); planned simulation platform, live trading platform and backtester (**all Proprietary**). |

The labels identify selected distribution licenses; SUL 1.0 means Sustainable Use License 1.0
and is source available. Existing repository license notices currently remain proprietary.
Licensing follows each component's intended distribution
and knowledge content, independently of its architectural layer. See the
[repository license register](master-plan/licensing.md#repository-license-register).
