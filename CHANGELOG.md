# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.4.4] - 2026-11-09

### Fixed

- **Server never closed Connections for dead clients.** The server's own keepalive interval was below its idle timeout, so each PING it sent restarted its own idle timer. The idle timeout never fired and every Connection the server had ever accepted stayed open, holding around four file descriptors each. After a few hours of browser traffic, roughly 240 zombie Connections accumulated and hit the default file descriptor limit, producing `Too many open files` on socket pair creation. The server keepalive interval is now above the idle timeout. The client still sends keepalives every two seconds — that is the direction that keeps SSH and WebTransport sessions alive, and it is unchanged. When a client dies, its PINGs stop and the server's idle timer runs to completion.
- **Socket pair file descriptors leaked when a Connection closed abruptly.** The shutdown that should have closed the socket pair write side was gated behind a success check, so on transport errors it never ran and both descriptors were held until process exit. The shutdown is now issued unconditionally.
- **The relay idle timeout was only observed while the relay was still generating events.** Once both sockets in a relay had nothing left to write, the event loop stopped firing for it, so the deadline was never re-checked and the relay and its descriptors stayed in the table indefinitely. The main loop now sweeps every relay each poll cycle rather than waiting for events. The timeout value and the condition are unchanged.
- **Two failure paths in the stream-accept loop exited without notifying the event loop.** Socket pair creation failure and hand-off failure both returned early, bypassing the cleanup notification, so the bridge's descriptors stayed open until process exit. Both now fall through to the same cleanup used by every other exit path.
- **The stream-accept loop could block forever after an abnormal Connection close.** It only exited on a clean end-of-stream, which does not always arrive. It is now also gated on the per-Connection cancellation handle.
- **The server TUI did not refresh after first-run registration.** The wallet address and key fingerprint in the Identity panel were captured at startup, before the account was provisioned. Both are now re-read after successful registration and reflected immediately on first launch.

### Added

- **`CONN_HARD_IDLE_SECS` (default 300 seconds).** Per-Connection watchdog. Force-closes any Connection with no application traffic for five minutes, regardless of whether the transport layer ever reports the close.
- **`RELAY_IDLE_SECS` (default 60 seconds).** Per-relay idle timeout. A relay whose one side has closed but whose peer never sent FIN is torn down after the timeout.

### Changed

- **Connection cleanup is structural.** The watchdog, stream-accept loop for a Connection share one cancellation handle. Whichever of them detects termination first cancels the rest, and the two descriptors attached to the Connection are released on every termination path — clean close, watchdog, transport error, or sibling cancel.

### Deployment note

- **Raise the process file descriptor limit to 65536** (systemd `LimitNOFILE` or `ulimit -n`). The fixes above eliminate the FD leaks, but the default of 1024 is still insufficient headroom for a proxy under normal browser load. Without it, transient `EMFILE` errors can occur during a burst even in the absence of a leak.


## [0.4.3] - 2026-11-7

### Added

- Added HMAC-derived key separation for HTTP authentication, tunnel authentication, and tunnel encryption.
- Added signature type identifiers for system and client signatures.
- Fixed client tunnel Hello address-length validation, rejecting addresses outside the supported 1–255-byte range.

### Changed

- Modified signed RFC 1929 credential parsing and serialization.

## [0.4.2] - 2026-10-25

### Added

- **Terminal UI dashboard** (`ui` feature, on by default) showing bind address, wallet, key fingerprint, relay URL, live upload/download, connection path (direct/relay), uptime, suspend status, and the active IP checker list.
- **Configurable IP checker list** via `HOST_IP_CHECKERS_LIST` (comma-separated HTTPS URLs). Unset or empty → built-in defaults. Only `https://` entries are accepted; invalid entries are skipped with a warning. The list is parsed once at startup and shared between the IP monitor and the dashboard.
- Headless fallback when stdout is not a TTY, plus terminal restore on panic and normal exit.
- `AppState` / `TrafficStats` for lock-free live stats shared between the byte pump and the UI.
- Graceful iroh endpoint teardown via `CancellationToken` and explicit `peer.close().await`.

### Fixed

- Silenced the recurring `Endpoint dropped without calling Endpoint::close` warning on shutdown.

### Documentation

- README: platform support section (Unix `AF_UNIX` vs. Windows loopback TCP pair), `ui` feature flag, and `HOST_IP_CHECKERS_LIST`.

## [0.4.1] - 2026-10-23

### Added

- **Windows support** for the unified bridge module. Previously Unix-only.
- Windows `imp` implementation of `ClientSocket`, `PairMioEnd` (`mio::net::TcpStream`), `PairTokioEnd` (`tokio::net::TcpStream`), and `create_pair()`.
- `create_pair()` on Windows emulates `socketpair(2)` via a loopback TCP pair (`127.0.0.1:0`), with `TCP_NODELAY` enabled on both ends to keep loopback latency low.

### Changed

- Bridge module is now cross-platform; `ClientSocket`, `PairMioEnd`, `PairTokioEnd`, `create_pair()`, and `pump_bi()` expose the same public surface on Unix and Windows.
- `create_pair()` is now `#[cfg]`-gated: `#[cfg(unix)]` uses `AF_UNIX` socket pair, `#[cfg(windows)]` uses loopback TCP.
- README updated to reflect Windows support.

## [0.4.0] - 2026-10-22

### Added

- Iroh-based transport architecture for SOCKS5 client-server communication.
- Encrypted QUIC session transport through Iroh.
- Direct peer-to-peer connectivity with relay fallback where supported by the network environment.
- Configurable Iroh endpoint bind addresses.
- Integration points for endpoint identity and application-level session authorization.
- Documentation covering architecture, connectivity, security, and deployment considerations.
- Attribution and project history linking to the original SOCKS5 Tunnel repository.

### Changed

- Extended the original SOCKS5 proxy architecture with Iroh-based endpoint connectivity.
- Separated the local SOCKS5 interface from the underlying client-server transport.
- Updated the project documentation to describe direct connectivity, relay fallback, and the application security model.

### Security

- Documented the distinction between transport encryption, peer identity, and application-level authorization.
- Documented the security implications of exposing an Iroh UDP endpoint.
- Clarified that transport encryption does not provide end-to-end confidentiality against the remote proxy server.
- Documented the importance of keeping the local SOCKS5 listener restricted to loopback unless explicitly configured otherwise.

### Documentation

- Added the project overview and architecture diagram to README.md.
- Added a SOCKS5 usage example using curl.
- Added configuration and firewall guidance.
- Documented the relationship with the original SOCKS5 implementation.

[Unreleased]: https://github.com/cryptoishere/iroh-socks5/compare/v0.4.4...HEAD
[0.4.4]: https://github.com/cryptoishere/iroh-socks5/releases/tag/v0.4.4
[0.4.3]: https://github.com/cryptoishere/iroh-socks5/releases/tag/v0.4.3
[0.4.2]: https://github.com/cryptoishere/iroh-socks5/releases/tag/v0.4.2
[0.4.1]: https://github.com/cryptoishere/iroh-socks5/releases/tag/v0.4.1
[0.4.0]: https://github.com/cryptoishere/iroh-socks5/releases/tag/v0.4.0
