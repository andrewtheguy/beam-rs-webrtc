# xfer-webrtc

> [!NOTE]
> This project is still work in progress (0.0.x). No backward compatibility is guaranteed between versions.

A secure, cross-platform, single-binary peer-to-peer file transfer tool built on encrypted WebRTC data channels.

## Features

- **Direct peer-to-peer transfer** — file bytes flow directly between peers over a WebRTC data channel; no server stores or relays your data.
- **End-to-end encrypted** — all traffic is carried over WebRTC's built-in DTLS encryption.
- **Single static binary** — one native executable per platform, no runtime, package manager, or dependencies to install.
- **Cross-platform** — Linux, macOS, and Windows.
- **Two signaling modes** — online via auto-discovered Nostr relays, or fully offline manual copy-paste (no internet or third party required).
- **Files and folders** — folders are auto-detected and archived (tar) before transfer.
- **Resumable transfers** — interrupted file transfers can resume from where they stopped, with checksum verification of the partial data.
- **NAT traversal** — STUN-based ICE negotiation establishes a direct connection across most networks.
- **Replay protection** — xfer codes and manual offers carry a timestamp and expire after a TTL.

## Installation

The release installers fetch a native, standalone executable. You only need the binary in your PATH; no runtime dependencies or package managers are required.

### Quick Install (Linux & macOS)

```bash
curl -sSL https://andrewtheguy.github.io/xfer/install.sh | bash
```

By default the installer pulls the latest **stable** release. Use `--prerelease` for the newest prerelease, or pass an explicit tag to pin to a specific build. Examples:

```bash
# Latest prerelease
curl -sSL https://andrewtheguy.github.io/xfer/install.sh | bash -s -- --prerelease

# Pin to a specific tag
curl -sSL https://andrewtheguy.github.io/xfer/install.sh | bash -s 20251210172710
```

### Quick Install (Windows)

```powershell
irm https://andrewtheguy.github.io/xfer/install.ps1 | iex
```

By default the PowerShell installer pulls the latest **stable** release. Use `-PreRelease` for the newest prerelease, or pass an explicit tag to pin to a specific build. Examples (args-only parser):

```powershell
# Latest prerelease
$env:XFER_INSTALL_ARGS='-PreRelease'; irm https://andrewtheguy.github.io/xfer/install.ps1 | iex

# Pin to a specific tag
$env:XFER_INSTALL_ARGS='20251210172710'; irm https://andrewtheguy.github.io/xfer/install.ps1 | iex
```

### From Source

```bash
cargo build --release
```

## Usage

Transfers use a **Xfer Code** to establish the connection. The code carries
signaling metadata for the WebRTC session.

### Online (Nostr signaling)

The default mode uses Nostr relays for signaling. Relays are auto-discovered
unless you specify custom Nostr relay URLs.

```bash
# Send a file
xfer-webrtc send /path/to/file

# Send a folder (auto-detected and archived)
xfer-webrtc send /path/to/folder

# Use the built-in default relays instead of auto-discovery
xfer-webrtc send --default-relays /path/to/file

# Use a custom Nostr relay URL (repeat --relay for multiple)
xfer-webrtc send --relay wss://relay1.example.com --relay wss://relay2.example.com /path/to/file
```

Receiving:

```bash
# Receive with the code from the sender
xfer-webrtc receive <XFER_CODE>

# Or prompt for the code interactively
xfer-webrtc receive

# Receive into a specific directory
xfer-webrtc receive <XFER_CODE> --output /path/to/dir

# Disable resumable transfers (don't save partial downloads)
xfer-webrtc receive <XFER_CODE> --no-resume
```

### Manual Mode (offline signaling)

Use manual mode when Nostr relays are unavailable and both peers still have
direct network reachability (for example, same LAN or a routed private/VPN
path). Offer/answer codes are exchanged by copy-paste.

```bash
# Sender
xfer-webrtc send --manual /path/to/file

# Receiver (auto-detects the manual offer when you paste it)
xfer-webrtc receive
```

The receiver uses the same `receive` command for both modes: paste a xfer code
for a normal Nostr transfer, or paste a manual offer code and it is detected
automatically.

## Common Use Cases

- **Send over the internet without exchanging IPs** — use the default online mode; Nostr relays handle signaling while the file flows directly peer-to-peer.
- **No internet or relays blocked** — use manual mode (`send --manual` / `receive`) to exchange signaling by copy-paste over a LAN or routed private/VPN network.
- **Send a whole directory** — pass a folder path; it is auto-detected and archived before transfer.

See [USE_CASES.md](docs/USE_CASES.md) for detailed scenarios.

For protocol details and wire formats, see [ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Security

- **Encrypted transport** — the WebRTC data channel runs over DTLS, so all file data is encrypted in transit, with ICE consent checks verifying connectivity for the life of the session.
- **No data on third parties** — Nostr relays and manual codes carry only signaling metadata (transfer ID, sender pubkey, relays, file metadata); file bytes never traverse a relay.
- **Replay and staleness protection** — xfer codes and manual offers include a creation timestamp and expire after a 60-minute TTL (with a small allowance for clock skew), and are validated before the handshake.
- **Integrity verification** — transfers carry a checksum so the receiver can detect corruption, including when resuming a partial download.

For the detailed security model, see [ARCHITECTURE.md](docs/ARCHITECTURE.md#security-model).

## License

MIT
