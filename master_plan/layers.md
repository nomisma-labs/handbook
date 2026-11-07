# Layers and integration boundaries

Updated 7 November 2026.

Nomisma separates foundational libraries, reusable runtime services, and applications. Each
layer owns both its components and the integration code needed to use them together. This
document records the agreed ownership boundaries; the component roadmaps describe implementation
status in the [foundation](foundation-layer.md), [service](service-layer.md), and
[application](application-layer.md) layers.

## Layer dependencies

Arrows mean **depends on**. The diagram shows the intended architecture, including planned
execution, multithreading, and simulation facilities.

```mermaid
flowchart TD
    apps["Application layer<br/>MD capture, replay and trading workflows"]
    services["Service layer<br/>Communication, execution and multithreading<br/>Timers, simulation, asio_service and app_terminator"]
    foundation["Foundation layer<br/>base, assembly, configuration and logging<br/>Integration between these libraries"]
    asio["Boost.Asio<br/>Live service backend"]

    apps --> services
    apps --> foundation
    services --> foundation
    services --> asio
```

| Layer | Owns | Examples |
| --- | --- | --- |
| Foundation | Common definitions, component composition, configuration, logging, and adapters between them | Configuration types, assembly/relay, JSON mapping for logger configuration |
| Service | Reusable runtime facilities provided to applications and integration between those facilities | `asio_comm`, timers, execution engines, multithreading, simulation, `asio_service`, `app_terminator` |
| Application | Selection and composition of services, domain behavior, configuration values, and operational policy | Exchange subscriptions, capture output, replay inputs, and conditions for successful completion |

Dependencies go toward lower layers. Foundation does not depend on service components or
Boost.Asio. Execution contracts belong to the service layer even when they are independent of
the backend. A simulation engine and scheduler therefore belong alongside the live runtime
services, with their own virtual-time implementation.

`cmake_template` supplies shared build infrastructure to all three layers through the copied
and synchronized kit. It is not a runtime dependency.

## Foundation integration

The planned `foundation` repository will combine `assembly`, `json_config`, and logging, using
`base` for shared definitions. It will own adapters between these components and select a tested
set of their versions. The component libraries retain their own repositories and APIs; they do
not depend back on the integrating `foundation` repository.

The repository has not yet been created. The distinction between intended ownership and current
source locations matters for the following examples.

### Logger configuration and JSON/C++ mapping

There are three separate responsibilities:

| Responsibility | Owner | Definition |
| --- | --- | --- |
| Configuration value types and presence policies | `base` | `nom.config_types`, including `cfg_defaulted`, `cfg_optional`, strings, and numeric types |
| Logger configuration structure, defaults, and validation | `spd_logger` | `log_level`, `logfile_config`, and `log_config` in `core/spd_logger/spd_logger.cppm`; validation in `core/spd_logger/log_config.cpp` |
| JSON/C++ mapping for those logger types | Planned `foundation` adapter, using `json_config` | Enum-name mapping and member mappings defined with `NOM_JCONFIG_ENUM` and `NOM_JCONFIG` |

For example, `spd_logger` exports this C++ configuration in namespace `nom`:

```cpp
struct log_config
{
    cfg_defaulted<log_level> level{log_level::info};
    cfg_optional<logfile_config> file{};

    std::expected<void, std::string> validate() const;
};
```

`logfile_config` contains the filename and optional rotation size. The default log level is
`info`; an unset file selects console output. These definitions support direct C++ construction
and validation without a JSON dependency in `spd_logger`.

`json_config` supplies generic mapping machinery through `json_config/config.h` and
`nom.json_config`. The adapter imports the configuration and logging modules and defines the
logger mappings in namespace `nom`, alongside the types for argument-dependent lookup:

```cpp
NOM_JCONFIG(logfile_config, name, rotation_size)
NOM_JCONFIG(log_config, level, file)
```

The adapter also uses `NOM_JCONFIG_ENUM` for `log_level`. The current mapping accepts lowercase
and uppercase names, emits lowercase names, and rejects unknown values. The macros provide
the JSON conversions and configuration-type registration; the field wrappers retain their
defaulted and optional behavior.

This adapter currently lives in the `md_capture` working tree at
`apps/okx/okx_md_capture/log_config.h`. Its agreed destination is `foundation`; the C++ types and
validation remain in `spd_logger`, and the generic mapping macros remain in `json_config`.

The application owns its `log` section and actual settings. The OKX application's
`app_config` contains `cfg_defaulted<nom::log_config> log`, includes that member in its own
`NOM_JCONFIG` mapping, and passes the loaded configuration to `nom::init_logger`. Moving the
adapter gives applications one shared logger mapping while preserving application ownership
of configuration values and startup.

## Services supplied to applications

`asio_service` and `app_terminator` both belong in the service layer. Their current definitions
are in `asio_comm/apps/common_app_nodes.h`, with a locally adapted copy in the `md_capture` working tree at
`apps/okx/okx_md_capture/common_app_nodes.h`. Placement under an application directory is their
current implementation location, not their intended architectural ownership. The shared
service-layer repository/module destination remains to be selected.

### asio_service

`asio_service` is an assembly node whose `context_mixin` contributes a
`boost::asio::io_context` to the application's context. Other nodes use that context and its
executor for asynchronous operations. Assembly constructs the context before the nodes and
destroys it after them.

The node supplies the execution context; the application drives `io.run()`. It does not create
worker threads. Future service-layer facilities can provide thread management or a simulation
engine. Assembly itself remains responsible for composition and synchronous relay dispatch.

### app_terminator

`app_terminator` provides termination coordination. It contributes an optional exit code to the
assembly context, handles `SIGINT` and `SIGTERM`, observes completion/failure events, and uses a
timer for configured runtime limits. It dispatches `service_command::stop` once. The
`md_capture` version additionally handles `feed_event::stop_failed`, allowing a subsequent
shutdown failure to change the exit code to failure.

Individual services perform their cleanup in their stop handlers. The current assembly places
`asio_service` first, then `app_terminator`, followed by the application nodes. This makes the
context available and lets the terminator forward stop requests to later nodes. The application
keeps the assembly alive while outstanding I/O callbacks finish.

The termination mechanism is a reusable service. The application supplies its configuration and
reports the events that express its outcome. Currently, `exit_after_sec`, `require_completion`,
and `feed_event` provide that connection: the timer starts on connection, and completion and
failure events determine the result. Those event and configuration interfaces can be reviewed
as the service is generalized; `app_terminator` remains a service supplied to the application.
