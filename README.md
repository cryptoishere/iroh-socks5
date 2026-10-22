# iroh-socks5

An authenticated SOCKS5 proxy powered by Iroh, with encrypted QUIC transport, direct peer-to-peer connectivity, and relay fallback.

iroh-socks5 extends the original SOCKS5 Tunnel implementation with an Iroh-based transport architecture for secure client-server communication.

The project combines a local SOCKS5 interface with authenticated Iroh sessions, encrypted transport, peer-to-peer connectivity where network conditions permit, and relay fallback when a direct connection cannot be established.

## Features

- **Standard SOCKS5 interface** — compatible with applications that support SOCKS5 proxies.
- **Local proxy endpoint** — exposes the client proxy on `127.0.0.1:1080`.
- **Encrypted QUIC transport** — uses Iroh to establish encrypted communication between endpoints.
- **Peer-to-peer connectivity** — attempts direct connections when network topology and NAT behavior permit.
- **Relay fallback** — maintains a connectivity path when direct peer-to-peer communication is unavailable.
- **Endpoint identity** — uses Iroh endpoint identities to identify remote peers.
- **Application-level authorization** — supports integration with account-bound credentials and authenticated session initialization.
- **Server-side destination connections** — the remote server connects to the requested target on behalf of the client.
- **Configurable endpoint binding** — allows endpoint bind addresses to be configured for different network environments.

## Architecture

```text
Application / Browser
        |
        | SOCKS5
        v
Client: 127.0.0.1:1080
        |
        | Authenticated Iroh session
        | Encrypted QUIC transport
        |
        +---- Direct peer-to-peer path
        |            or
        +---- Iroh relay fallback
                     |
                     v
             Remote SOCKS5 server
                     |
                     | Outbound connection
                     v
              Target destination
```

The client provides a local SOCKS5 endpoint. Applications connect to that endpoint without needing to understand Iroh or the underlying transport.

The client establishes a session with the remote server. Iroh handles endpoint connectivity and encrypted transport, attempting a direct peer-to-peer path when possible and using a relay when necessary.

The remote server processes the proxied request and establishes the connection to the requested destination.

## Quick example

Once the client is running, use `curl` to send a request through the local SOCKS5 proxy:

```bash
curl -v --proxy socks5h://127.0.0.1:1080 https://smartholdem.io
```

The `socks5h` scheme requests proxy-side hostname resolution, so the destination hostname is resolved through the proxy rather than locally by curl.

The client must be running and configured to connect to the intended remote endpoint before this command will work.

## Connection model

### Direct peer-to-peer connectivity

Iroh attempts to establish direct connectivity between endpoints when network conditions permit. This can avoid routing application traffic through a relay.

Successful direct connectivity depends on factors including NAT behavior, endpoint address discovery, routing, and firewall policy.

### Relay fallback

When a direct path cannot be established or maintained, Iroh can use a configured relay to carry traffic between endpoints.

Direct connectivity is not guaranteed on every network. If direct peer-to-peer connectivity is a deployment requirement, verify the negotiated connection path at runtime instead of assuming that a successful session is direct.

## Endpoint binding and firewall considerations

The Iroh endpoint bind address is separate from the local SOCKS5 listener.

- The local SOCKS5 listener should normally bind to `127.0.0.1:1080`.
- The Iroh endpoint uses a UDP socket for QUIC transport.
- Direct connectivity may require inbound UDP reachability, NAT traversal, and appropriate firewall configuration.
- Opening a UDP port can improve direct connectivity in some environments, but it does not guarantee that every NAT or firewall configuration will permit a direct connection.
- Relay connectivity can remain available when direct connectivity is blocked, provided the relay is reachable and configured.
- Do not expose the local SOCKS5 listener to public interfaces unless remote access is explicitly required and properly secured.

## Security model

### Transport encryption

Iroh provides encrypted transport for communication between endpoints. This protects data in transit between the client and remote server.

### Peer identity and authorization

An Iroh endpoint identity identifies a peer, but identity alone does not grant permission to use the proxy. The server should independently validate the client's authorization and account-bound credentials during session establishment.

### Application-level security

Authentication, account binding, credential lifecycle, revocation, and access control are responsibilities of the application protocol and its integration with the account system.

### Privacy and destination traffic

The remote proxy server necessarily processes the SOCKS5 requests and destination connections. Transport encryption does not prevent the server from seeing data after it has been decrypted.

Use HTTPS or another end-to-end encrypted protocol to protect application content from the proxy operator and intermediate systems beyond the encrypted client-server transport.

### Network exposure

A reachable Iroh UDP endpoint is not automatically an open SOCKS5 proxy. Nevertheless, exposed endpoints can receive unsolicited packets and connection attempts. Keep authentication and authorization enforced, apply resource limits where appropriate, and restrict network access when practical.

## Configuration

Endpoint settings should be configured for the deployment environment.

Depending on the application configuration, relevant settings may include:

| Setting | Purpose |
|---|---|
| `ENDPOINT_BIND_ADDRESS` | Local address and port used by the remote Iroh endpoint |
| `RELAY_URL_PUBLIC` | Configured relay URL |

The exact environment variables and defaults depend on the binary and configuration implementation. Consult the relevant configuration code and deployment documentation before deploying.

Do not assume that a bind address, advertised endpoint address, and public NAT-mapped address are interchangeable.

## Project history

The project builds upon the original SOCKS5 Tunnel repository, which implemented a minimal SOCKS5 proxy in Rust using mio, non-blocking I/O, SOCKS5 protocol framing, authentication, and server-side DNS resolution.

iroh-socks5 extends that foundation with an Iroh-based transport architecture, separating the local proxy interface from remote endpoint connectivity and encrypted session transport.

See the original repository for the earlier implementation and project history.

## Development status

This README describes the intended architecture and integration points. Actual feature availability, configuration names, and security guarantees depend on the current implementation.

Verify direct connectivity, relay fallback, authentication enforcement, and firewall behavior in the deployed environment before relying on them in production.

## License

See the repository's `LICENSE` file for applicable licensing terms.
