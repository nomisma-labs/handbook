# Nomisma Labs — Statement of Intent

1. **Purpose.** Nomisma Labs is a development space for a new generation of trading platforms. The goal is to build a software workshop: reusable components, tools, and practices for constructing a family of trading systems that meet different business and technical requirements.

2. **Relation to other trading platforms.** The Nomisma platform is inspired by our experience with earlier crypto trading platforms, Napoleon and Axiom. Nomisma does not reuse source code from either platform.

3. **System composition.** Each reusable component is developed in a separate repository, with its boundaries shaped by the needs of future systems. Each trading system will have a repository that brings together the components required for its use case. For example, an HFT system may require a dedicated performance testing framework, while a mid-frequency system may not. Separate component repositories support different combinations, access rights, licenses, and release cycles without keeping all projects in one monorepo.

4. **Development and distribution.** Components will have a private development version under a proprietary license, with the full development and agent tooling, skills, internal documentation, comprehensive tests, and CI/CD. Where distributed, a public or client version will carry an appropriate open-source or commercial license, with reduced documentation, test coverage, and CI/CD suited to its audience. Components intended solely for internal tooling, such as `cmake_template`, will remain private.

5. **Dependencies.** Repositories will depend on one another through Git submodules and explicit build-system dependencies. Shared components such as `config` and `logger` will support many repositories; each system repository will assemble a compatible set of component versions.

## Decisions

- **License:** private repositories start under a proprietary license, with `LICENSE` at the root. Copyright belongs to Alexey Tkachenko until transferred to Nomisma Labs. Distribution licenses are chosen per component.
- **SPDX:** every private Nomisma source file carries `SPDX-FileCopyrightText: 2026 Alexey Tkachenko` and `SPDX-License-Identifier: LicenseRef-Nomisma-Labs-Proprietary`.
- **Hosting:** GitHub organization, private repositories by default, GitHub Actions for CI/CD; start on the Free plan.
- **Language:** module-first C++, using the latest stable standard supported by the project toolchain. Use `nom` as the root namespace, component namespaces where useful, and standard-library-style `snake_case` naming.

The initial foundational repositories and their implementation details are covered in [foundation-layer.md](master_plan/foundation-layer.md).
