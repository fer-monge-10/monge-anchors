# monge-anchors

Public, append-only **Merkle anchors** for the Monge platform's append-only signals
ledger. Each anchor is a timestamped commitment to the state of the ledger at a
point in time, published here so that any later rewrite of the (private) ledger
becomes detectable.

## What an anchor file contains

Each `anchor_YYYYMMDDTHHMMSSZ.json` records:

- `computed_at`  - UTC timestamp the anchor was computed.
- `n_systems`    - number of systems included.
- `n_signals`    - number of signal rows (leaves) covered.
- `per_system`   - one entry per system: `{system_key, count, head_hash}`
                   (the system's signal count and the hash of its latest signal).
- `merkle_root`  - the Merkle root over every signal hash, ordered by
                   `(system_key, seq)`.
- `algo`         - `"sha256-v1"`.

**Merkle construction (`sha256-v1`):** leaves are the signals' hash values (hex
sha256); two children combine as `sha256(left_32_bytes || right_32_bytes)`; a level
with an odd number of nodes duplicates its last node; the root is the hex digest.

## Why it exists

The Monge ledger is append-only and hash-chained: every signal's hash depends on the
previous one, so changing any past signal changes every hash after it, up to that
system's `head_hash`. Publishing the Merkle root to this public, timestamped,
append-only repository is a commitment: if the private ledger is ever rewritten, its
recomputed root will no longer match a previously published anchor, and the first
divergent system is identifiable. This is the external tamper-evidence layer (the
"F4" mitigation) referenced in the platform's decision log.

## Honest limitation (current)

Today the platform repository and API are **private**, so these anchors are
**public commitments by the operator** rather than independently reproducible proofs:
a third party cannot yet recompute the root themselves because they cannot read the
underlying signal hashes. What they CAN rely on now is the public, timestamped,
append-only history of roots in this repo - the operator cannot silently backdate or
alter a commitment already pushed here.

Independent third-party recomputation becomes possible when the platform's public
read surface ships (a public endpoint or export of signal hashes). At that point
anyone will be able to fetch the hashes, recompute the root, and check it against the
anchor published here for the corresponding timestamp.
