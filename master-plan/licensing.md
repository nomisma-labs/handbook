# Licensing and ownership

Decided 11 November 2026.

Copyright belongs to **Alexey Tkachenko**. This document records the licensing policy from the
[memorandum](../memorandum.md) for Nomisma components and their distributed products.

## Licensing hierarchy

| Tier | Components | Distribution license |
| --- | --- | --- |
| General low-level building blocks | Common types, composition, configuration, ordinary logging and general core integration | **MIT**, the default for the core layer |
| Domain knowledge | Repositories containing reusable domain models, workflows or domain-specific integration | **Sustainable Use License 1.0**, as source-available software |
| Distinctive know-how | Components whose implementation embodies know-how retained privately, including `trace_logger` | **Proprietary** |

Architectural placement and licensing are separate decisions. `trace_logger` remains a core
component and is explicitly proprietary. Membership in `nom_core` does not determine a
component's license; neither does being outside that facade.

## Repository license register

These are the selected **distribution licenses** used in the repository descriptions throughout
this master plan. The existing repositories currently retain proprietary `LICENSE` files;
MIT and SUL 1.0 labels record the intended distribution terms, not completed relicensing.

| Repository or planned component | Distribution license | Scope |
| --- | --- | --- |
| [`base`](core-layer.md#4-base-common-definitions) | **MIT** | Common definitions and utilities |
| [`assembly`](core-layer.md#5-assembly--integration-framework) | **MIT** | Composition, relay and delegates |
| [`json_config`](core-layer.md#json_config) | **MIT** | JSON configuration framework |
| [`spd_logger`](core-layer.md#spd_logger) | **MIT** | General-purpose application logging |
| [`nom_core`](core-layer.md#nom_core-facade) | **MIT** | Core facade and reusable core integration |
| [`toml_config`](core-layer.md#toml_config) | **MIT** | TOML configuration framework; outside `nom_core` |
| [`yaml_config`](core-layer.md#yaml_config) | **MIT** | YAML configuration framework; outside `nom_core` |
| [`trace_logger`](core-layer.md#trace_logger) | **Proprietary** | High-performance logging know-how; outside `nom_core` |
| [`asio_comm`](service-layer.md#communication-asio_comm) | **Sustainable Use License 1.0** | Communication services and exchange-connectivity integration |
| Planned `nom_services` | **Sustainable Use License 1.0** | Shared runtime and service integration; repository not yet created |
| [`md_capture`](application-layer.md#market-data-capture--md_capture) | **Sustainable Use License 1.0** | Market-data capture workflows and domain policy |
| [Planned simulation platform](application-layer.md#components-and-current-status) | **Proprietary** | Trading workflows in a controlled simulation environment; repository not yet defined |
| [Planned live trading platform](application-layer.md#components-and-current-status) | **Proprietary** | Live market data, strategy decisions and order execution; repository not yet defined |
| [Planned backtester](application-layer.md#components-and-current-status) | **Proprietary** | Historical strategy evaluation and analysis; repository not yet defined |
| [`cmake_template`](toolchain-layer.md#current-implementation-cmake_template) | **Proprietary** | Private development and build tooling |
| [Planned performance measurement framework](core-layer.md#8-performance-measurement-framework) | **MIT** | General core measurement facilities; repository and API remain to be defined |

Future repositories need their own entry under this hierarchy. Unnamed future facilities do
not yet have a repository-level license assignment.

## MIT distributions

MIT is the chosen open-source license. It permits broad reuse, modification and distribution,
including commercial use, subject to retaining the required copyright and permission notices.
Use the [MIT license text](https://opensource.org/license/mit) with Alexey Tkachenko as the
copyright holder for Nomisma-owned code.

## Sustainable Use License 1.0 distributions

Use the [Sustainable Use License, Version 1.0](https://github.com/n8n-io/n8n/blob/master/LICENSE.md#sustainable-use-license)
for the selected domain-knowledge components. Its terms permit internal business use and
non-commercial or personal use; providing the software to others is limited to free,
non-commercial distribution, with notice and modification requirements. The full license text
governs the permissions and restrictions.

Describe this tier as **source available**. Its restrictions on use distinguish it from an
open-source license, as explained in the [license author's documentation](https://github.com/n8n-io/n8n-docs/blob/main/docs/n8n-community-license/README.md#is-n8n-open-source).
Nomisma's repository scope and copyright notices must identify its own components and holder;
the reference project's product-specific enterprise exclusions do not define Nomisma's scope.

## Development repositories and delivered products

Private development repositories can retain proprietary tooling, internal documentation,
experiments and other material beyond the distributed product. A product release selects its
contents and includes the chosen MIT, SUL 1.0 or proprietary terms. Public visibility alone does
not identify which license applies.

This plan records the decisions; it does not replace existing repository `LICENSE` files or
relicense published revisions. When preparing a distribution, align its license text, source
notices and package metadata with the selected tier. Preserve the licenses and notices of
third-party dependencies; a facade does not replace its dependencies' terms.

The [packaging plan](wip/packaging.md) covers source, native and Python delivery formats.
The license choice applies to the distributed component across those formats.

## Open distribution decisions

- **General runtime services versus domain knowledge.** `asio_comm` and planned `nom_services`
  currently select SUL 1.0, but their transport and lifecycle facilities also fit the broad
  description of reusable low-level infrastructure. The plan needs a more precise boundary
  between MIT infrastructure and SUL domain integration, including whether a distribution
  separates those parts. The repository selections above remain in place pending that decision.
- **Build support in distributed source.** Derived repositories contain copies of the proprietary
  `cmake_template` kit. Decide which build-support files accompany MIT/SUL products and under
  what terms, or provide alternative build support. Keeping the template repository private
  does not define the redistribution terms of copies included with a product. This remains
  part of preparing the delivered contents and notices described in the packaging plan.
