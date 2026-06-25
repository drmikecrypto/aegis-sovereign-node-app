# Sovereign node suite — threat model (operator + user)

**Scope:** Headless CLI HTTP services in `sovereign-node-app/cli/` (JSON-RPC proxy + static circuit hosting). Desktop/mobile companions inherit the same trust boundaries once they embed the same services.

---

## 1. What the suite does **not** provide

| Claim | Reality |
|-------|---------|
| “Private Sonic” | You still submit transactions to **public** Sonic validators; inclusion, ordering, and calldata are **unchanged**. |
| “Hidden mempool” | The proxy forwards `eth_sendRawTransaction` to configured **upstreams** — it is not a sealed builder or encrypted mempool. |
| “Hides wallet IP” | Your browser or wallet still talks to **your** node; **your** node’s operator sees IP and payloads. Tor/VPN is a separate layer. |
| “Trustless circuits” | Optional SHA-256 integrity in config is **operator-supplied**; users must verify hashes out-of-band (release notes, signed manifests). |

---

## 2. Trust assumptions

| Component | Trust |
|-----------|--------|
| **`upstreams` RPC URLs** | Whoever runs those endpoints sees forwarded JSON-RPC (reads + signed tx bytes on send). |
| **Operator of this machine** | Full visibility into logs (if enabled), config, disk, and process memory. |
| **TLS** | Default examples use **HTTP on loopback** (`127.0.0.1`). Binding to `0.0.0.0` without TLS exposes RPC to LAN — **avoid** unless you add TLS or firewall rules. |
| **Circuit files on disk** | Supply-chain: replace `circuits/` tree only via governed / checksummed releases. |

---

## 3. Hardening checklist

- [x] Keep **`listen`** on **`127.0.0.1`** — enforced when `security.requireLoopbackBind` is true (default).
- [x] **Local-only RPC** by default — public upstreams stripped unless `security.allowPublicRpcUpstreams`.
- [x] **Circuit path traversal** blocked; optional SHA-256 integrity (see `docs/OPERATIONAL_SECURITY.md`).
- [x] **Redacted upstream errors** in operational profile.
- [x] **No `shell.open`** in Tauri capabilities.
- [ ] Prefer **HTTPS upstreams** when public fallback is enabled; rotate keys if `config.json` leaks.
- [ ] Run `npm run circuits:manifest` after each release bundle.
- [ ] Document for users: **sovereign node shifts trust from public RPC vendor to whoever runs the node** (often the user — good; sometimes a DAO — disclose).

---

## 4. Related docs

- Monorepo [`docs/OMNICHAIN_PRIVACY_AND_RPC.md`](../../docs/OMNICHAIN_PRIVACY_AND_RPC.md)
- [`docs/STACK_ALIGNMENT.md`](../../docs/STACK_ALIGNMENT.md) — port **8547**, env wiring
- [`../Aegis-contracts/docs/ops/PRIVACY_UX_LOCAL_STORAGE_AND_DEVICE.md`](../Aegis-contracts/docs/ops/PRIVACY_UX_LOCAL_STORAGE_AND_DEVICE.md) — browser-side storage (orthogonal to RPC)

Last updated: 2026-06-12 (Phase D).
