# Application layer: Market-data capture and trading workflows

Updated 12 November 2026.

The [memorandum](../memorandum.md#architecture-directory) places applications above the
[service layer](service-layer.md). Applications depend on services and obtain
[core components](core-layer.md) transitively through those dependencies. An application
composes services into an operational workflow and owns configuration, deployment, lifecycle
policy and the interpretation of results.

## Components and current status

| Component | Distribution license | Role | Current work |
| --- | --- | --- | --- |
| [md_capture](https://github.com/nomisma-labs/md_capture) | **SUL 1.0** | Capture exchange market data for analysis and replay. | Under development. |
| Simulation platform | **Proprietary** | Compose runtime services and simulation models to run trading workflows in a controlled environment. | TBD |
| Live trading platform | **Proprietary** | Compose live market data, strategy decisions and order execution with application lifecycle and operational policy. | TBD |
| Backtester | **Proprietary** | Evaluate strategies against historical data and produce results for comparison and analysis. | TBD |

Distribution licenses follow the [licensing plan](licensing.md).

## Market Data Capture : md_capture

`md_capture` collects market data and symbol information from exchanges and retains it for
historical analysis and replay.

- **Location:** the [md_capture repository](https://github.com/nomisma-labs/md_capture),
  at `md_capture/` in the workspace root.
- **License:** SUL 1.0 is the selected distribution license; the current repository notices
  remain proprietary.
- **Direct repository dependency:** [asio_comm](service-layer.md#communication-asio_comm),
  pinned as the sole external submodule at `external/asio_comm`. Core libraries are supplied
  transitively through that dependency.
