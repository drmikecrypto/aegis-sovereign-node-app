# Operational security profile

How a high-assurance team would ship the Aegis native app — and what we implemented.

## CIA-style principles vs. our old defaults

| Concern | Old / convenience | Operational (default now) |
|--------|-------------------|---------------------------|
| **RPC upstreams** | Round-robin to public Sonic RPC | **Local node only** (`127.0.0.1:8545`); public URLs stripped unless opted in |
| **Network bind** | Loopback (good) | **Enforced** — refuses `0.0.0.0` / LAN bind |
| **Upstream errors** | Full error text to UI | **Redacted** — no provider hostnames leaked to WebView |
| **RPC methods** | Allowlist | Same allowlist (no `debug_*`, no `personal_*`) |
| **Circuit files** | Static serve | **Path traversal blocked** + optional **SHA-256 pins** |
| **Logging** | `info` | **`warn`** in operational profile; hyper/reqwest muted |
| **Tauri shell** | `shell:allow-open` | **Removed** — compromised UI cannot spawn external URLs |
| **Release binary** | Standard | **LTO + strip** |
| **Failover** | Round-robin | **Sequential** — try local first, never fan reads across providers |

## Config files

| File | Use |
|------|-----|
| `shared/config.example.json` | **Operational** default (ship this posture) |
| `shared/config.convenience.json` | Dev/demo: public RPC fallback allowed |
| `shared/config.json` | Operator copy (gitignored) |

Copy convenience only on a dev machine:

```powershell
copy sovereign-node-app\shared\config.convenience.json sovereign-node-app\shared\config.json
```

## Pin circuit hashes (supply chain)

After `npm run bundle`:

```powershell
node sovereign-node-app\shared\scripts\generate-circuit-manifest.mjs
```

Sets `circuits.integrity.enabled: true` and SHA-256 for every `.wasm` / `.zkey`. Mismatched files return **409** — prover cannot load trojaned artifacts.

## What this does **not** do

- Tor/VPN (operator adds separately)
- Encrypted mempool / private submission
- Hide traffic from Sonic validators
- Stop a compromised OS or physical access

See `THREAT_MODEL.md` for full trust boundaries.

## Mobile / desktop

Same Rust engine on all platforms. Android cleartext to loopback is required for local RPC; upstream policy still follows `security.allowPublicRpcUpstreams`.
