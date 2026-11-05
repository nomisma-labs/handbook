# Application layer: Market-data capture and trading workflows

Updated 5 November 2026.

The [memorandum](../memorandum.md#architecture-directory) places applications above the
[foundation](foundation-layer.md) and [service](service-layer.md) layers. An application selects
compatible component versions and turns their APIs into an operational workflow. It owns
configuration, deployment, lifecycle policy and the interpretation of results.

## First component: md_capture

`md_capture` is the first planned application component: capture exchange market data using
the communication services and retain it for analysis and replay. This is application work
to begin next; the current `asio_comm` feeders are examples, not its implementation.

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
component integration, and foundation components for configuration and logging. Capture-specific
policy belongs here or in its exchange adapters; protocol mechanics remain in the service layer.

Repository layout, storage choices and the first supported exchange set remain design decisions.
The JSON configuration repository's `md_capture` example is a configuration demonstration,
not a released capture application. The existing feeder/provider examples can inform integration
without becoming a shared exchange-client abstraction prematurely.

## Growth beyond capture

Further applications can compose the same foundations and services for replay, private-feed
processing and order execution. They are future scope, not delivered components of this layer.
Multithreading facilities will belong to the service layer; each application will choose its
execution topology and the ownership policy for data crossing those boundaries.
