# Sovereign Node Companion – Architecture

## Goals

- **Zero middlemen:** user runs their own Sonic RPC locally or via hardware they control.
- **One-click experience:** download → press **Start** → node boots, RPC exposed at `127.0.0.1:8545`.
- **Immutable UX:** desktop/mobile apps re-use the same audited UI bundle we publish to Arweave.
- **Deterministic releases:** every binary, manifest, and checksum is reproducible and signed by governance.
- **Domain parity:** the embedded UI always resolves to the same Arweave transaction as `https://aegiscoin.sonic`, verified via governance-controlled manifests.

## High-Level Flow

```
[User Action]          [Companion App]                [Sonic Node]
      │                       │                               │
      │ Launch app            │                               │
      ├──────────────────────>│                               │
      │                       │ Download & verify binary      │
      │                       ├──────────────┬───────────────>│
      │                       │              │                │
      │                       │ Checksum sig │                │
      │                       │ (shared manifest)             │
      │                       └──────────────┴───────────────<│
      │                       │ Spawn node process (RPC on 8545)
      │                       ├──────────────────────────────>│
      │                       │ Proxy UI to http://127.0.0.1:8545
      │<──────────────────────┤                               │
      │ Browser/WebView loads local RPC                       │
      │                       │                               │
```

## Components

### 1. Desktop Companion (Tauri)
- Rust shell controls node lifecycle (start/stop, logs).
- WebView hosts the existing Vite build; assets bundled locally for offline availability.
- IPC channel exposes node state (synced, peering, RPC health) to the UI.
- Auto-updater pulls new Sonic binaries when manifest changes.

### 2. Mobile Companion (Flutter)
- Uses platform-specific services (foreground service on Android, background task on iOS) to keep the node running when UI minimized.
- Embeds the same UI via Flutter WebView (or ported components) pointing at the local RPC (Loopback on mobile).
- Offers Tor/mixnet integration hook for future privacy layers.

### 3. Headless CLI
- Minimal Node.js/TypeScript or Rust script that installs/verifies/runs the Sonic node as a managed process or systemd service.
- Shares manifest logic with desktop/mobile to ensure the same binary set.

### 4. Shared Manifest (`shared/binaries.json`)
- Lists available Sonic versions, download URLs, SHA256 hashes, signatures, and supported platforms.
- Governance signs manifest updates before release.
- Companion apps trust only entries signed by the governance key and verify that the bundled UI hash matches the Arweave transaction referenced by the latest timelocked governance proposal.

## Sonic Node Expectations

- Follow Sonic Labs’ node deployment guide for flags, pruning, and RPC configuration [[1]](https://docs.soniclabs.com/).
- Default runtime flags:
  - `--http --http.api eth,net,web3`
  - `--http.addr 127.0.0.1`
  - `--http.port 8545`
  - `--syncmode full` (light sync research ongoing)
  - Optional: `--port 30303` (p2p), `--ws` for WebSocket RPC if needed.
- Companion app can expose advanced settings (custom data dir, peer count limits) in an “Expert” drawer.

## Security Considerations

- All downloads verified via SHA256 + governance signature before execution.
- Node process runs in restricted directories; no arbitrary file writes outside app data.
- UI warns if RPC is exposed beyond localhost to prevent accidental public nodes.
- Logs sanitized to avoid leaking secrets.
- Future work: integrate proof-carrying RPC responses / light client once Sonic offers it.

## Next Steps

1. Wire Tauri skeleton (`desktop/`): embed UI, create Rust command to spawn Sonic binary.
2. Author binary manifest & signature tooling in `shared/`.
3. Prototype CLI wrapper for manual testing.
4. Mirror design for Flutter and evaluate resource footprint.
5. Define release pipeline: GitHub Actions → Arweave/IPFS asset publish → governance checksum announcement.


