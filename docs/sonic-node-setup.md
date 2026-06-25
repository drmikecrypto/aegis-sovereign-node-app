# Sonic Node Bootstrap Reference

Authoritative checklist for preparing Sonic binaries used by all companion apps.

## Requirements

- 8+ vCPU, 32 GB RAM, NVMe SSD
- 1 Gbps network (preferred)
- Linux x86_64 (primary target); add arm64 builds as Sonic releases permit

## Build Steps

1. Fetch the official Sonic source or binary per Sonic Labs documentation [[1]](https://docs.soniclabs.com/).
2. Verify upstream checksums and signatures.
3. Produce our deterministic build artifacts:
   - Strip binaries
   - Generate SHA256 checksum
   - Sign with DAO release key
4. Populate `shared/binaries.json` with:
   ```json
   {
     "version": "v2025.11.07",
     "platform": "linux-x86_64",
     "url": "https://<mirror>/sonic-v2025.11.07-linux-x86_64.tar.gz",
     "sha256": "...",
     "signature": "0x...",
     "released_at": "2025-11-07T15:00:00Z"
   }
   ```

## Runtime Flags

```
sonicd \
  --datadir <app-data>/sonic \
  --http --http.addr 127.0.0.1 --http.port 8545 \
  --http.api eth,net,web3 \
  --ws --ws.addr 127.0.0.1 --ws.port 8546 --ws.api eth,net,web3 \
  --syncmode full \
  --cache 4096 \
  --port 30303 \
  --metrics --metrics.addr 127.0.0.1 --metrics.port 6060
```

Companion apps expose only a simplified toggle; advanced flags live behind an “Expert” accordion.

**Public RPC fallbacks (copy into `rpc.upstreams` as needed):** Sonic mainnet **146** — `https://rpc.soniclabs.com`; Sonic testnet **14601** — `https://rpc.testnet.soniclabs.com` ([Getting Started](https://docs.soniclabs.com/sonic/build-on-sonic/getting-started)).

## Health Checks

- `eth_syncing` → false (synced)
- `eth_blockNumber` increases
- `net_peerCount` within expected range
- `eth_chainId` equals 146 or 14601 depending on build

## Publishing

1. Upload archives + manifest to Arweave/IPFS.
2. Publish signed checksum list on-chain via governance proposal (Timelock enforces activation delay).
3. Update `shared/binaries.json` and commit to repo, ensuring the UI hash matches the Arweave transaction pinned to `https://aegiscoin.sonic`.


