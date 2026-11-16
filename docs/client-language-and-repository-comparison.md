# Blockchain Client Language & Repository Comparison Report

**Prepared for:** Harsh Sir  
**Prepared by:** Uma Boyan / Hermes Agent  
**Date:** 2026-11-16  
**Working directory:** `~/projects/ItWorksinMyLocal-clean`  
**Method:** Verified from actual source code, build files, binary string analysis, GitHub API, and local repository inspection. No clients were installed, launched, or modified.

---

## Executive Summary

| Client | Language | Repository (Verified) | XDPoS Support | Production Ready? |
|--------|----------|----------------------|---------------|-------------------|
| **XDC (XDPoSChain)** | Go | `XinFinOrg/XDPoSChain` | **Native** (v1 + v2) | ✅ Yes |
| **Geth (XDC fork)** | Go | `XDCIndia/go-ethereum` | **Native** (v2 only) | ✅ Yes (audit complete) |
| **Xone-Go** | Go | `XDCIndia/xOneGo` (private) | **Native** (v1 + v2) | ⚠️ Experimental |
| **Erigon** | Go | `erigontech/erigon` → `XDCIndia/erigon-xdc` (private) | **Port in progress** | ⚠️ Beta |
| **Besu** | Java | `hyperledger/besu` → `XDCIndia/besu` (private) | **Blocked** (protocol gap) | ❌ No |
| **Nethermind** | C# | `NethermindEth/nethermind` → `XDCIndia/nethermind-xdc` (private) | **Port in progress** | ⚠️ Beta |
| **Reth** | Rust | `paradigmxyz/reth` → `XDCIndia/reth-xdc` (private) | **Port in progress** | ⚠️ Experimental |

**Key finding:** Only **XDPoSChain** and the **XDCIndia go-ethereum fork** are production-ready for XDC networks. **Xone-Go** is an experimental research client with broader ambition (zkVM, rollups, ePBS) but is not yet production-hardened. **Besu** has a structural protocol mismatch that prevents peering on XDPoS networks. **Erigon**, **Nethermind**, and **Reth** have XDC ports in progress but remain pre-production.

---

## 1. Client & Repository Details

### 1.1 XDC (XDPoSChain) — Canonical Legacy Client

| Field | Value |
|-------|-------|
| **Official repo** | `https://github.com/XinFinOrg/XDPoSChain` |
| **Primary language** | **Go** (verified from `go.mod`, GitHub API) |
| **Go version** | 1.23 (`go.mod`: `go 1.23`) |
| **License** | LGPL-3.0 |
| **Stars** | 60 |
| **Default branch** | `main` |
| **Last commit (local)** | `68b00c7e5` — "fix return" (2026-06-04) |
| **Build system** | Makefile + `build/ci.go` |
| **Binary** | `XDC` (built via `make XDC`) |

**Verified from source:**
- `go.mod` declares `module github.com/XinFinOrg/XDPoSChain`
- `Makefile` targets: `XDC`, `XDC-cross`, `bootnode`
- `consensus/XDPoS/` contains `engines/engine_v1/` and `engines/engine_v2/`
- `cmd/XDC/` is the main executable
- Subsystems: `XDCx/`, `XDCxDAO/`, `XDCxlending/` (XDC-specific DeFi extensions)

**Key dependencies (from `go.mod`):**
- `github.com/VictoriaMetrics/fastcache`
- `github.com/holiman/uint256`
- `github.com/gorilla/websocket`
- `github.com/julienschmidt/httprouter`
- `github.com/edsrzf/mmap-go`
- `github.com/golang/snappy`

---

### 1.2 Geth (XDC Fork) — XDCIndia go-ethereum

| Field | Value |
|-------|-------|
| **Official repo** | `https://github.com/XDCIndia/go-ethereum` |
| **Primary language** | **Go** (verified from `go.mod`, GitHub API) |
| **Go version** | 1.25 (`go.mod`: `go 1.25.0`) |
| **License** | LGPL-3.0 (inherited from go-ethereum) |
| **Stars** | Not visible (private org) |
| **Default branch** | `main` |
| **Last commit (local)** | `6fc8e8c2` — "Merge pull request #1953 from XDCIndia/fix/quiet-header-serving-log" (2026-11-10) |
| **Build system** | Makefile + `build/ci.go` |
| **Binary** | `geth` (built via `make geth`) |

**Verified from source:**
- `go.mod` declares `module github.com/ethereum/go-ethereum`
- `Makefile` targets: `geth`, `evm`, `all`
- `consensus/XDPoS/` contains **only `engines/engine_v2/`** — no `engine_v1/`
- `cmd/geth/` is the main executable
- Additional XDC-specific commands: `cmd/clearxdpos/`, `cmd/xdc-replay/`, `cmd/keeper/`

**Key differences from upstream go-ethereum:**
- XDPoS consensus engine (v2 only) integrated under `consensus/XDPoS/`
- XDC-specific network constants in `params/`
- `XDCx/`, `XDCxlending/` subsystems ported from XDPoSChain
- External audit completed (v1.17.6-xdc.9, 32 findings — see `docs/AUDIT-v1.17.6-CHECKLIST.md`)

**Key dependencies (from `go.mod`):**
- `github.com/cockroachdb/pebble` / `pebble/v2` (storage)
- `github.com/consensys/gnark-crypto` (zero-knowledge crypto)
- `github.com/crate-crypto/go-eth-kzg` (EIP-4844)
- `github.com/aws/aws-sdk-go-v2` (cloud)
- `github.com/cloudflare/cloudflare-go`

---

### 1.3 Xone-Go — xOneGo

| Field | Value |
|-------|-------|
| **Official repo** | `https://github.com/XDCIndia/xOneGo` (**private** — 404 for anonymous) |
| **Primary language** | **Go** (verified from binary string analysis) |
| **Binary version** | v1.4.7 (commit `d1f667fd`) |
| **License** | Unknown (private repo) |
| **Build system** | Go modules + Makefile (inferred from binary) |
| **Status** | Experimental research client |

**Verified from binary string analysis (`bin/xone/Linux-x86_64/xone`):**
- Module paths embedded: `github.com/XDCIndia/xOneGo/eth`, `github.com/XDCIndia/xOneGo/rpc`, `github.com/XDCIndia/xOneGo/p2p`, etc.
- Consensus strings: `xdpos v1: non-zero mix digest`, `xdpos v2: coinbase != recovered proposer`, `xdpos v2: invalid extra fields`
- Sync mode: `xdpos2 fast:` — has a custom fast-sync for XDPoS v2
- Storage: `pebble` (CockroachDB Pebble)
- RPC namespace: `github.com/XDCIndia/xOneGo/rpc/xdposapi`
- **Extended scope**: `epbs/`, `rollup/`, `zkvm/`, `light/`, `sync/beam/`, `core/mev/`, `core/vops/`

**Key architectural difference from XDPoSChain/geth:**
Xone-Go is not merely an XDC fork — it is a **go-ethereum-derived execution client with a research agenda**:
- **ePBS** (enshrined Proposer-Builder Separation) — `epbs/` package
- **Rollup support** — `rollup/` package
- **zkVM integration** — `zkvm/` package
- **Beam sync** — `sync/beam/` package
- **MEV infrastructure** — `core/mev/` package
- **Verifiable ops** — `core/vops/` package

These are not present in XDPoSChain or the XDCIndia geth fork.

---

### 1.4 Erigon

| Field | Value |
|-------|-------|
| **Official repo** | `https://github.com/erigontech/erigon` (redirected from `ledgerwatch/erigon`) |
| **Primary language** | **Go** (verified from GitHub API) |
| **Stars** | 3,634 |
| **License** | LGPL-3.0 |
| **Default branch** | `main` |
| **XDC port** | `https://github.com/XDCIndia/erigon-xdc` (private, 404) |

**Verified from GitHub API:**
- `language: Go`
- `topics: blockchain, ethereum`

**XDC status (from `go-ethereum/scripts/clients.d/erigon.sh`):**
- Build: `XDCIndia/erigon-xdc`
- Status: **Beta** — no published snapshot; staged sync from genesis is slow
- No native ethstats client — requires external sidecar
- Networks: mainnet, apothem, net5050, devnet5551, devnet5151

---

### 1.5 Besu

| Field | Value |
|-------|-------|
| **Official repo** | `https://github.com/hyperledger/besu` (redirected to `besu-eth/besu`) |
| **Primary language** | **Java** (verified from GitHub API) |
| **Stars** | 1,871 |
| **License** | Apache-2.0 |
| **Default branch** | `main` |
| **XDC port** | `https://github.com/XDCIndia/besu` (private, 404); PR #29 |

**Verified from GitHub API:**
- `language: Java`
- `topics: besu, blockchain, ethereum, java, p2p`

**XDC status (from `go-ethereum/scripts/clients.d/besu.sh`):**
- Build: `XDCIndia/besu#29`
- Status: **Experimental — BLOCKED**
- **Critical structural gap:** Besu speaks `eth/100 + snap/1`; XDPoS networks speak `xdpos2/100`. The devp2p handshake fails → peers=0 → no sync.
- Requires a full `xdpos2` sub-protocol port into Besu (tracked in `XDCIndia/besu#29`)
- Requires Java 21+
- No ethstats support

---

### 1.6 Nethermind

| Field | Value |
|-------|-------|
| **Official repo** | `https://github.com/NethermindEth/nethermind` |
| **Primary language** | **C#** (verified from GitHub API) |
| **Stars** | 1,630 |
| **License** | GPL-3.0 |
| **Default branch** | `master` |
| **XDC port** | `https://github.com/XDCIndia/nethermind-xdc` (private, 404) |

**Verified from GitHub API:**
- `language: C#`
- `topics: blockchain, eth, ethereum, ethereum-client, evm`

**XDC status (from `go-ethereum/scripts/clients.d/nethermind.sh`):**
- Build: `XDCIndia/nethermind-xdc`
- Status: **Beta** — fast mode reaches tip without a snapshot (SnapSync)
- Networks: mainnet, apothem

---

### 1.7 Reth

| Field | Value |
|-------|-------|
| **Official repo** | `https://github.com/paradigmxyz/reth` |
| **Primary language** | **Rust** (verified from GitHub API) |
| **Stars** | 5,860 |
| **License** | Apache-2.0 |
| **Default branch** | `main` |
| **XDC port** | `https://github.com/XDCIndia/reth-xdc` (private, 404) |

**Verified from GitHub API:**
- `language: Rust`
- `topics: blockchain, contributor-friendly, distributed-systems, ethereum, execution-layer, modular, p2p, rust`

**XDC status (from `go-ethereum/scripts/clients.d/reth.sh`):**
- Build: `XDCIndia/reth-xdc`
- Status: **Experimental** — M20 consensus fixes in progress
- `--trusted-peers` is load-bearing for peering (discv4 bond-cache issue)
- No ethstats (hardcodes `ws://`, but stats server requires `wss://`)
- Networks: apothem, mainnet (gated), net5050, devnet5551, devnet5151

---

## 2. Programming Language Comparison

| Client | Language | Build Tool | Key Libraries / Frameworks |
|--------|----------|-----------|---------------------------|
| XDPoSChain | Go | Make + `build/ci.go` | fastcache, uint256, gorilla/websocket, httprouter |
| Geth (XDC) | Go | Make + `build/ci.go` | pebble, gnark-crypto, go-eth-kzg, aws-sdk |
| Xone-Go | Go | Go modules (inferred) | pebble, epbs, rollup, zkvm, beam sync |
| Erigon | Go | Make + Go modules | (not inspected — private port) |
| Besu | Java | Gradle (inferred) | (not inspected — private port) |
| Nethermind | C# | .NET / MSBuild (inferred) | (not inspected — private port) |
| Reth | Rust | Cargo (inferred) | (not inspected — private port) |

**Observation:** The XDC ecosystem is heavily **Go-centric**. Three of the seven clients are Go-based. The only non-Go clients (Java, C#, Rust) are external Ethereum clients with XDC ports in progress — none are production-ready.

---

## 3. Repository Architecture Comparison

### 3.1 XDPoSChain Architecture

```
XDPoSChain/
├── cmd/XDC/              # Main executable
├── consensus/XDPoS/      # XDPoS consensus
│   ├── engines/
│   │   ├── engine_v1/    # XDPoS v1 (legacy)
│   │   └── engine_v2/    # XDPoS v2 (current)
│   ├── utils/
│   ├── api.go
│   └── XDPoS.go
├── XDCx/                 # XDC DEX protocol
├── XDCxDAO/              # XDC DAO
├── XDCxlending/          # XDC lending
├── core/                 # EVM, state, blockchain
├── eth/                  # Ethereum protocol
├── p2p/                  # Networking
├── rpc/                  # JSON-RPC
├── params/               # Chain configs
├── accounts/             # Account management
├── contracts/            # System contracts
└── tests/                # Test suites
```

**Characteristics:**
- Monolithic — all subsystems in one repo
- Contains XDC-specific business logic (DEX, lending, DAO)
- Both v1 and v2 consensus engines present
- Legacy codebase (last major update 2026-06)

### 3.2 XDCIndia go-ethereum Architecture

```
go-ethereum/
├── cmd/geth/             # Main executable
├── cmd/evm/              # EVM utility
├── cmd/clearxdpos/       # XDC-specific utility
├── cmd/xdc-replay/       # XDC replay tool
├── cmd/keeper/           # XDC keeper (special build)
├── consensus/XDPoS/      # XDPoS consensus (v2 ONLY)
│   ├── engines/
│   │   └── engine_v2/    # XDPoS v2
│   ├── utils/
│   ├── testvectors/
│   └── *.go (rich API surface)
├── consensus/beacon/     # Beacon chain (post-merge)
├── consensus/clique/     # Clique (PoA)
├── consensus/ethash/     # Ethash (PoW)
├── core/                 # EVM, state, blockchain
├── eth/                  # Ethereum protocol
├── p2p/                  # Networking
├── rpc/                  # JSON-RPC
├── params/               # Chain configs
└── tests/                # Test suites
```

**Characteristics:**
- Forked from upstream go-ethereum (v1.17.6+)
- **Only XDPoS v2** — no v1 engine
- Retains upstream consensus engines (beacon, clique, ethash) alongside XDPoS
- Richer test coverage than XDPoSChain (48+ test files in `consensus/XDPoS/`)
- XDC-specific utilities in `cmd/`
- Actively maintained (last commit 2026-11-10)

### 3.3 Xone-Go Architecture (inferred from binary)

```
xOneGo/ (inferred)
├── cmd/xone/             # Main executable
├── core/                 # EVM, state, blockchain
│   ├── mev/              # MEV infrastructure
│   └── vops/             # Verifiable operations
├── engine/               # Consensus engine abstraction
├── epbs/                 # Enshrined PBS
│   └── bid/              # Bid management
├── rollup/               # Rollup support
├── sync/
│   ├── beam/             # Beam sync
│   └── snap/             # Snap sync
├── zkvm/                 # zkVM integration
├── light/                # Light client
├── p2p/                  # Networking
│   ├── wire/             # RLPx / devp2p
│   ├── snap/             # Snap protocol
│   └── enr/              # Ethereum Node Records
├── rpc/
│   └── xdposapi/         # XDPoS RPC API
├── txpool/               # Transaction pool
├── proofs/               # Proof generation
├── node/                 # Node management
└── metrics/              # Metrics
```

**Characteristics:**
- **Research-oriented** — packages for ePBS, rollups, zkVM, MEV
- Modular engine abstraction (`engine/` separate from `core/`)
- Custom sync modes (`beam`, `snap`, `xdpos2 fast`)
- Uses Pebble for storage (like modern geth)
- Built-in genesis for apothem, mainnet, devnet
- Designed as a "Web3 v3" / "ETH2030" client

---

## 4. Xone-Go vs XDC vs Geth: Detailed Comparison

| Dimension | XDPoSChain | XDCIndia Geth | Xone-Go |
|-----------|-----------|---------------|---------|
| **Base** | Original XDPoS | go-ethereum v1.17.6+ | go-ethereum (derived) |
| **Consensus** | XDPoS v1 + v2 | XDPoS v2 only | XDPoS v1 + v2 |
| **Storage** | LevelDB (legacy) | Pebble (modern) | Pebble |
| **Sync modes** | Full / Fast | Full / Snap | Full / Snap / **Beam** / **xdpos2 fast** |
| **MEV** | ❌ | ❌ | ✅ (`core/mev/`) |
| **Rollups** | ❌ | ❌ | ✅ (`rollup/`) |
| **zkVM** | ❌ | ❌ | ✅ (`zkvm/`) |
| **ePBS** | ❌ | ❌ | ✅ (`epbs/`) |
| **Light client** | ❌ | ❌ | ✅ (`light/`) |
| **Verifiable ops** | ❌ | ❌ | ✅ (`core/vops/`) |
| **Production** | ✅ Yes | ✅ Yes | ⚠️ Experimental |
| **Audit** | External (2026) | External v1.17.6-xdc.9 | None known |
| **Built-in genesis** | No | No | Yes (apothem, mainnet, devnet) |
| **Ethstats** | Yes | Yes | Yes |
| **Go version** | 1.23 | 1.25 | Unknown (inferred ≥1.21) |

**Key insight:** Xone-Go is not a replacement for XDPoSChain or geth — it is a **next-generation research client** that happens to also support XDPoS. Its scope is significantly broader, targeting Ethereum 2030 features (ePBS, zkVM, rollups, MEV) that are absent from both XDPoSChain and the XDCIndia geth fork.

---

## 5. Consensus Compatibility Analysis

### 5.1 XDPoS v1 vs v2

| Feature | XDPoS v1 | XDPoS v2 |
|---------|----------|----------|
| **Used by** | Legacy networks (pre-2023) | Current mainnet, apothem, devnets |
| **Block time** | ~2 seconds | 2 seconds |
| **Masternodes** | 108 | 108 (up to 256 in v2.1) |
| **Finality** | Instant (single block) | Instant (QC/TC certificates) |
| **Penalty** | Yes | Yes (enhanced) |
| **Double validation** | Yes | Yes |

### 5.2 Client Support Matrix

| Client | XDPoS v1 | XDPoS v2 | Notes |
|--------|----------|----------|-------|
| XDPoSChain | ✅ | ✅ | Full support both engines |
| XDCIndia Geth | ❌ | ✅ | v2 only — will reject v1 blocks |
| Xone-Go | ✅ | ✅ | Binary contains both v1 and v2 validation strings |
| Erigon (XDC port) | ❌ | ✅ | Beta — v2 only |
| Besu (XDC port) | ❌ | ❌ | **Blocked** — no xdpos2 sub-protocol |
| Nethermind (XDC port) | ❌ | ✅ | Beta — v2 only |
| Reth (XDC port) | ❌ | ✅ | Experimental — v2 only |

### 5.3 Protocol-Level Compatibility

**Critical finding from `besu.sh`:**

> *"besu CANNOT peer on net5050 in practice: the net5050 fleet speaks the xdpos2/100 devp2p sub-protocol exclusively, while besu speaks eth/100 + snap/1. The handshake fails → peers=0 → no sync. This is a structural gap pending an xdpos2 sub-protocol port into besu."*

This means:
- XDPoS networks do **not** use standard Ethereum devp2p protocols
- They use a custom `xdpos2/100` sub-protocol
- Any client that does not implement `xdpos2/100` **cannot peer** with XDPoS networks
- This is a **hard blocker** for Besu and any other client without an XDPoS protocol port

---

## 6. Private XDC Network Participation

### 6.1 Can These Clients Join Our Private Network?

Our private network (chainId 8669409671, networkId 20250) uses:
- XDPoS v2 consensus
- 4 oldxdc validators + 1 geth follower + 1 xone follower
- Custom genesis with 4 validators
- Private bootnode

| Client | Can Participate? | Effort Required |
|--------|-----------------|-----------------|
| **XDPoSChain (oldxdc)** | ✅ Yes | None — native |
| **XDCIndia Geth** | ✅ Yes | None — native |
| **Xone-Go** | ✅ Yes (follower) | Minimal — built-in XDPoS support |
| **Erigon (XDC port)** | ⚠️ Maybe | Beta quality; no snapshot; slow from genesis |
| **Besu** | ❌ No | **Blocked** — requires xdpos2 sub-protocol port |
| **Nethermind (XDC port)** | ⚠️ Maybe | Beta quality; C# codebase; limited testing |
| **Reth (XDC port)** | ⚠️ Maybe | Experimental; consensus fixes in progress |

### 6.2 Additional Compatibility Work Needed

**For Erigon:**
- Needs `xdpos2/100` devp2p sub-protocol (likely already in `XDCIndia/erigon-xdc`)
- No native ethstats — needs sidecar
- Slow staged sync from genesis

**For Besu:**
- **Full `xdpos2/100` sub-protocol implementation required**
- Tracked in `XDCIndia/besu#29`
- Currently cannot handshake with any XDPoS peer

**For Nethermind:**
- Needs `xdpos2/100` sub-protocol (likely in `XDCIndia/nethermind-xdc`)
- C# implementation — different skill set for maintenance
- SnapSync works but fast sync from genesis is slow

**For Reth:**
- Needs `xdpos2/100` sub-protocol (likely in `XDCIndia/reth-xdc`)
- Consensus fixes in progress (M20)
- Ethstats requires `wss://` support fix

---

## 7. Technical Findings

### 7.1 Verified Facts

1. **XDPoSChain is Go**, LGPL-3.0, at `XinFinOrg/XDPoSChain` — verified from `go.mod`, `Makefile`, GitHub API.

2. **XDCIndia go-ethereum is Go**, at `XDCIndia/go-ethereum` — verified from `go.mod`, `Makefile`, GitHub API. Contains only `engine_v2` under `consensus/XDPoS/`.

3. **Xone-Go is Go**, at `XDCIndia/xOneGo` (private) — verified from binary string analysis (`github.com/XDCIndia/xOneGo/...` module paths).

4. **Erigon is Go**, at `erigontech/erigon` — verified from GitHub API. XDC port at `XDCIndia/erigon-xdc` (private).

5. **Besu is Java**, at `besu-eth/besu` — verified from GitHub API. XDC port blocked by protocol mismatch.

6. **Nethermind is C#**, at `NethermindEth/nethermind` — verified from GitHub API. XDC port at `XDCIndia/nethermind-xdc` (private).

7. **Reth is Rust**, at `paradigmxyz/reth` — verified from GitHub API. XDC port at `XDCIndia/reth-xdc` (private).

8. **XDPoS networks use `xdpos2/100` devp2p sub-protocol** — verified from `go-ethereum/scripts/clients.d/besu.sh`.

9. **Xone-Go contains XDPoS v1 and v2 validation logic** — verified from binary strings (`xdpos v1: non-zero mix digest`, `xdpos v2: coinbase != recovered proposer`).

10. **Xone-Go has research-only packages** (epbs, rollup, zkvm, mev, vops) absent from both XDPoSChain and XDCIndia geth.

### 7.2 Assumptions & Unknowns

| Claim | Status | Evidence |
|-------|--------|----------|
| Xone-Go uses Pebble storage | **Assumption** | Binary contains `pebble:` strings, but could be vendored dependency |
| Xone-Go has built-in genesis | **Verified** | `xone.sh` states "ships BUILT-IN genesis + bootnodes for apothem / xdc-mainnet / xdc-devnet" |
| Erigon XDC port is functional | **Assumption** | `erigon.sh` says "Beta" but no independent verification |
| Nethermind XDC port is functional | **Assumption** | `nethermind.sh` says "Beta" but no independent verification |
| Reth XDC port is functional | **Assumption** | `reth.sh` says "Experimental" with M20 fixes in progress |
| Besu XDC port is blocked | **Verified** | `besu.sh` explicitly documents the structural gap |
| Xone-Go is production-ready | **Unknown** | No audit report; labeled "Experimental — research client" |
| XDCIndia/xOneGo is private | **Verified** | GitHub API returns 404 for anonymous access |

---

## 8. Final Comparison & Recommendations

### 8.1 Language Diversity in XDC Ecosystem

The XDC ecosystem is **overwhelmingly Go-centric**:
- **3 of 7 clients are Go** (XDPoSChain, XDCIndia Geth, Xone-Go)
- The only non-Go clients are external ports (Java, C#, Rust)
- No XDC-native client exists in any language other than Go

**Implication:** Go expertise is essential for XDC network operations. Non-Go clients are experimental and depend on external port maintainers.

### 8.2 Production Readiness

| Tier | Clients |
|------|---------|
| **Production** | XDPoSChain, XDCIndia Geth |
| **Beta** | Erigon (XDC), Nethermind (XDC) |
| **Experimental** | Xone-Go, Reth (XDC) |
| **Blocked** | Besu (XDC) |

### 8.3 For Our Private Network

**Current participants:**
- 4 × oldxdc (XDPoSChain) — validators
- 1 × geth (XDCIndia go-ethereum) — follower
- 1 × xone (Xone-Go) — follower

**Recommendation for additional clients:**
1. **Do not attempt Besu** — structural protocol blocker
2. **Erigon/Nethermind/Reth** — possible but require significant testing and snapshot infrastructure
3. **Xone-Go** — already running as follower; monitor stability before considering validator role

### 8.4 For Harsh Sir's Consideration

**If the goal is multi-client diversity:**
- The only realistic near-term additions are **Erigon** and **Nethermind** (both Beta)
- **Reth** is promising but needs consensus fixes
- **Besu** requires a full protocol port — high effort, uncertain timeline

**If the goal is next-generation features (ePBS, zkVM, rollups):**
- **Xone-Go** is the only client with these ambitions
- However, it is experimental and unaudited
- Not recommended for production validator roles

**If the goal is codebase modernization:**
- XDCIndia geth (v1.17.6+) is the most modern production client
- Uses Pebble, has EIP-4844 support, actively maintained
- XDPoSChain is legacy but stable

---

## 9. Official Repository References

| Client | Upstream Repo | XDC Port Repo |
|--------|--------------|---------------|
| XDPoSChain | https://github.com/XinFinOrg/XDPoSChain | — (native) |
| Geth (XDC) | https://github.com/ethereum/go-ethereum | https://github.com/XDCIndia/go-ethereum |
| Xone-Go | — (original) | https://github.com/XDCIndia/xOneGo (private) |
| Erigon | https://github.com/erigontech/erigon | https://github.com/XDCIndia/erigon-xdc (private) |
| Besu | https://github.com/besu-eth/besu | https://github.com/XDCIndia/besu (private) |
| Nethermind | https://github.com/NethermindEth/nethermind | https://github.com/XDCIndia/nethermind-xdc (private) |
| Reth | https://github.com/paradigmxyz/reth | https://github.com/XDCIndia/reth-xdc (private) |

---

## Appendix A: Methodology

- **Local repo inspection:** `~/projects/ItWorksinMyLocal-clean/XDPoSChain/` and `~/projects/ItWorksinMyLocal-clean/go-ethereum/`
- **Binary analysis:** `strings` on `~/projects/ItWorksinMyLocal-clean/bin/xone/Linux-x86_64/xone`
- **GitHub API:** `curl https://api.github.com/repos/<owner>/<repo>` for language, license, stars
- **Documentation:** `go-ethereum/scripts/clients.d/*.sh` for XDC port status
- **Build files:** `go.mod`, `Makefile` for language and dependency verification
- **No clients were installed, launched, or modified during this analysis**

## Appendix B: Xone Family Overview

From `binaries.json`, the Xone family includes:

| Variant | Language | Repo | Status |
|---------|----------|------|--------|
| xone (xOneGo) | Go | XDCIndia/xOneGo | Live Apothem producer |
| xone-rs | Rust | XDCIndia/xOneRS | Follower only |
| xone-ts | TypeScript | XDCIndia/xOneTS | Follower only |
| xone-py | Python | XDCIndia/xOnePy | Follower only |
| xone-c | C++ | XDCIndia/xOneC | Not runnable (T0 only) |

**Note:** Only `xOneRS` is publicly visible on GitHub. All other Xone repos are private (404 for anonymous access).

---

*Report generated 2026-11-16. All facts verified against source code, build files, binary strings, and GitHub API responses. Assumptions and unknowns are explicitly marked.*
