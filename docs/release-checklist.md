# Release Checklist – Sovereign Node Suite

## Pre-Release

- [ ] Run Sonic build pipeline (see `sonic-node-setup.md`).
- [ ] Update `shared/binaries.json` with new entries.
- [ ] Bump companion app versions (desktop/mobile/cli).
- [ ] Regenerate checksums and sign manifest.
- [ ] Update change log.

## Verification

- [ ] Desktop app boots node and UI on Windows / macOS / Linux.
- [ ] Mobile app starts node (foreground service) on Android & iOS.
- [ ] CLI installs/upgrades node on fresh machine.
- [ ] RPC health checks pass (`eth_blockNumber`, `net_peerCount`, `eth_chainId`).
- [ ] UI connects via `127.0.0.1:8545` and displays governance dashboard.

## Publish

- [ ] Upload binaries + manifest to Arweave/IPFS mirrors.
- [ ] Publish governance announcement with checksums & URLs.
- [ ] Tag repo (`sovereign-node-app@vx.y.z`).
- [ ] Update download links in web dApp.
- [ ] Update `aegiscoin.sonic` content hash to the new Arweave transaction and mirror in the sovereign-node manifest.
- [ ] Verify the Arweave frontend bundle hash matches `dist/manifest.hash.json` before embedding in the desktop/mobile WebView.


