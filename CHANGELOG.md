# Changelog

All notable changes to this project are documented in this file.

The format is based on Keep a Changelog, and this project adheres to Semantic Versioning.

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

[Unreleased]: https://github.com/cryptoishere/iroh-socks5/compare/v0.4.1...HEAD
[0.4.1]: https://github.com/cryptoishere/iroh-socks5/releases/tag/v0.4.1
[0.4.0]: https://github.com/cryptoishere/iroh-socks5/releases/tag/v0.4.0
