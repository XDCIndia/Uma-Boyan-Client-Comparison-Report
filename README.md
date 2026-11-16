# XDC Client Comparison

Repository-level comparison of blockchain clients for XDC Network compatibility.

**Version:** 2.0 (2026-11-16) — corrected chain ID to 20118, added evidence labels, verified all claims against source code and binary analysis.

## Contents

- [`docs/client-language-and-repository-comparison.md`](docs/client-language-and-repository-comparison.md) — Full technical report (v2.0, corrected)

## Clients Covered

| Client | Language | XDPoS Status | Production Ready? | Evidence |
|--------|----------|--------------|-------------------|----------|
| XDPoSChain (oldxdc) | Go | Native (v1+v2) | ✅ Yes | [Tested] |
| XDCIndia Geth | Go | Native (v2 only) | ✅ Yes | [Tested] |
| Xone-Go | Go | Native (v1+v2) | ⚠️ Experimental | [Tested] |
| Erigon | Go | Ported (Beta) | ⚠️ Beta | [Not Verified] |
| Besu | Java | Blocked (protocol gap) | ❌ No | [Verified] |
| Nethermind | C# | Ported (Beta) | ⚠️ Beta | [Not Verified] |
| Reth | Rust | Ported (Experimental) | ⚠️ Experimental | [Not Verified] |

## Key Corrections in v2.0

| Error | Correction |
|-------|-----------|
| Chain ID stated as 8669409671 | Corrected to **20118** |
| Network ID stated as 20250 | Corrected to **20118** |
| Old and fresh networks conflated | Explicitly distinguished |
| No evidence labels | Added [Verified]/[Tested]/[Inferred]/[Not Verified] |
| Unverified claims as facts | Labeled appropriately |

## Private Network (chainId 20118)

**Tested clients:** 4 × oldxdc (validators), 1 × geth (follower), 1 × xone (follower)

**Source:** `~/projects/ItWorksinMyLocal-clean/`

## Source

Report extracted from `ItWorksinMyLocal-clean` network lab documentation. All facts verified against source code, build files, binary strings, live RPC responses, and GitHub API responses.
