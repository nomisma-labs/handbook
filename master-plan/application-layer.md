# Application layer: Market-data capture and trading workflows

Updated 11 November 2026 from the current `md_capture` working tree, including staged
application-control and design changes. This is implementation status, not a release claim.

The [memorandum](../memorandum.md#architecture-directory) places applications above the
[service layer](service-layer.md). Applications depend on services and obtain
[core components](core-layer.md) transitively through those dependencies. An application
composes services into an operational workflow and owns configuration, deployment, lifecycle
policy and the interpretation of results.

## Components and current status

| Component | Distribution license | Role | Current work |
| --- | --- | --- | --- |
| [md_capture](https://github.com/nomisma-labs/md_capture) | **SUL 1.0** | Capture exchange market data for analysis and replay. | OKX Spot capture is implemented; broader source coverage and operational guarantees remain under development. |
| Simulation platform | **Proprietary** | Compose runtime services and simulation models to run trading workflows in a controlled environment. | Planned; repository, API and detailed scope remain to be defined. |
| Live trading platform | **Proprietary** | Compose live market data, strategy decisions and order execution with application lifecycle and operational policy. | Planned; repository, API and detailed scope remain to be defined. |
| Backtester | **Proprietary** | Evaluate strategies against historical data and produce results for comparison and analysis. | Planned; repository, API and detailed scope remain to be defined. |

The [licensing plan](licensing.md) selects **Sustainable Use License 1.0 (SUL 1.0)** for
`md_capture`, reflecting its capture workflows and domain policy. This is the source-available
distribution plan; the current repository still has proprietary license notices.
The planned simulation platform, live trading platform and backtester are **proprietary**.

## Market Data Capture : md_capture

**Distribution license: Sustainable Use License 1.0 (source available).**

[`md_capture`](https://github.com/nomisma-labs/md_capture) is the first application repository:
capture exchange market data using the communication services and retain it for analysis and
replay. It uses the [toolchain layer](toolchain-layer.md), currently `cmake_template`, and depends
on `asio_comm`, which supplies the core components through `nom_core`.
The current OKX Spot application captures received JSON into one configured append file,
with a local microsecond timestamp for each record. `nom::md_logger` owns the buffered payloads
and writes them on one worker thread; shutdown stops the producer, drains the writer and
reports writer failure. The `asio_comm` feeders remain connectivity examples, separate from
this recording application.

The current composition uses `config_owner`, `asio_service`, `app_control`, `md_sink`, and
`md_subscriber`. JSON configuration selects symbols, channels, transport settings, storage and
lifecycle limits. `app_control` consumes completion/failure responses and requests periodic
health checks; its runtime limit starts at application start. Reusable lifecycle mechanisms
belong in the service layer, while capture policy stays here.

The planned scope includes around 20 data sources. OKX Spot is the current reference; that
target breadth is not implemented capture coverage. Remaining work includes:

- Further exchange adapters, catalog discovery, snapshots and request-stream capture.
- Connection, subscription, application-heartbeat and reconnection policy, including detection
  of missing useful data and recording of gaps.
- Package, stream and session identity; binary framing and protocol context beyond JSON payloads.
- Process-owned package folders, exact hour/day rotation, bounded buffering/backpressure,
  final pending-record progress, crash recovery, durability and archive transfer.
- Independent backend reconciliation, symbol partitioning, normalization and retention, while
  realtime recording preserves source observations.
- Operational diagnostics and reproducible validation using local fixtures and live connectivity
  checks; readiness must be distinguished from merely receiving an acknowledgement.

The single-file writer has an unbounded backlog and does not establish power-loss durability.
Local storage and TLS integration tests exist; sustained operation, complete feed readiness,
gap accounting and recovery remain to be qualified. Protocol mechanics stay in `asio_comm`;
capture-specific policy belongs here or in the source adapters.

The implementation uses `apps/okx/okx_md_capture/` and `src/md_logger/`; an independent
`src/md_logger_ai/` implementation remains available for experimentation. The
[working-tree reference design](https://github.com/nomisma-labs/md_capture/blob/master/docs/md_capture/reference_design.md),
[improvement proposal](https://github.com/nomisma-labs/md_capture/blob/master/docs/md_capture/design_proposal.md)
and [work list](https://github.com/nomisma-labs/md_capture/blob/master/TODO.md) separate current
behavior from package storage, backend services and product plans.
The `md_capture` examples in `json_config`, [toml_config](core-layer.md#toml_config) and
[yaml_config](core-layer.md#yaml_config) demonstrate configuration; they are separate from
the capture application repository. The existing feeder/provider examples can inform
integration without becoming a shared exchange-client
abstraction prematurely.

## Open design decisions

The simulation platform and backtester can both involve historical replay and simulated
execution. Their product boundary is not defined yet: decide whether the backtester consumes
the simulation platform or independently composes shared services. Reusable execution engines,
virtual-time scheduling and thread management belong in the service layer; applications own
scenarios, strategy evaluation and interpretation of results. The three planned application
components retain their proprietary license selection.
