# Application layer: Market-data capture and trading workflows

Updated 5 November 2026.

The [memorandum](../memorandum.md#architecture-directory) places applications above the
[core](core-layer.md) and [service](service-layer.md) layers. An application selects
compatible component versions and turns their APIs into an operational workflow. It owns
configuration, deployment, lifecycle policy and the interpretation of results.

## First component: md_capture

[`md_capture`](https://github.com/nomisma-labs/md_capture) is the first application repository:
capture exchange market data using the communication services and retain it for analysis and
replay. Its initial scaffold comes from the
[toolchain layer](toolchain-layer.md), currently `cmake_template`, and pins `base`, `assembly`,
`spd_logger`, `asio_comm` and `json_config`. Capture applications and storage are not yet
implemented; the current `asio_comm` feeders remain integration examples.

The initial scope should define:

- Exchange/symbol/channel selection and catalog discovery through exchange adapters.
- Connection, subscription, application-heartbeat and reconnection policy, including detection
  of missing useful data and recording of gaps.
- Capture timestamps and source identity, with a decision on raw versus normalized records.
- Storage format, buffering/backpressure, retention and graceful shutdown that drains or
  explicitly accounts for pending output.
- Operational diagnostics and reproducible validation using local fixtures and live connectivity
  checks; readiness must be distinguished from merely receiving an acknowledgement.

The first composition will use `asio_comm` for transport and timers, Assembly/relay for
component integration, and core components for configuration and logging. Capture-specific
policy belongs here or in its exchange adapters; protocol mechanics remain in the service layer.

Exchange applications will live under `apps/<exchange>/<app>/`, with shared capture code added
under `src/` as its contracts emerge. Storage choices and the first supported exchange set
remain design decisions, tracked in the repository's [work list](https://github.com/nomisma-labs/md_capture/blob/master/TODO.md).
The JSON configuration repository's `md_capture` example is a configuration demonstration,
not a released capture application. The existing feeder/provider examples can inform integration
without becoming a shared exchange-client abstraction prematurely.

## Growth beyond capture

Further applications can compose the same core libraries and services for replay, private-feed
processing and order execution. They are future scope, not delivered components of this layer.
Multithreading facilities will belong to the service layer; each application will choose its
execution topology and the ownership policy for data crossing those boundaries.
