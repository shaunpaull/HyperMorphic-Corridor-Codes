# HyperMorphic-Corridor-Codes
State-adaptive erasure coding by invariant corridor recovery.


HyperMorphic Corridor Codes

State-adaptive erasure coding by invariant corridor recovery.

HyperMorphic Corridor Codes, or HCC, are a prototype coding-theory primitive where recovery depends not only on the number of surviving shards, but on whether those shards span a valid algebraic-geometric reconstruction corridor.

Traditional threshold codes ask:

Do we have at least k shards?

HCC asks:

Do the surviving shards preserve enough algebraic capacity, representation-lane coverage, angular spread, and state-dependent invariant span to reconstruct?

This means two shard subsets with the same cardinality can behave differently: one may recover, while another may fail because it is geometrically clustered.

⸻

Core Idea

Fixed threshold coding:

recoverable ⇔ |S| ≥ k

HyperMorphic Corridor Coding:

recoverable ⇔ capacity(S, state) is sufficient
              and lane_span(S, state) is sufficient
              and corridor_score(S, state) exceeds threshold

Where S is the surviving shard subset.

HCC replaces fixed k-of-n recovery with corridor-admissible recovery.

⸻

Why This Exists

Most erasure and threshold codes use fixed decoding rules:

* fixed block size
* fixed alphabet
* fixed threshold
* fixed reconstruction condition
* fixed coding geometry

HCC explores a different direction:

What if the decoding threshold itself is state-dependent?

The code adapts reconstruction requirements based on causal local entropy and geometric shard distribution.

Low-complexity regions may recover through smaller corridors.
High-complexity regions require stronger invariant span.

⸻

Features

* State-adaptive reconstruction
    * Required shard count changes with entropy/state level.
* Geometry-aware recovery
    * Shards are assigned to representation lanes.
    * Recovery requires sufficient lane coverage, not just shard count.
* CRT-based algebraic reconstruction
    * Uses pairwise-coprime adaptive prime moduli.
    * Reconstructs chunks using the Chinese Remainder Theorem.
* SafeGear-style modular base transform
    * Each prime modulus receives a coprime base.
    * Encoding uses modular winding:

r_i = (b_i · C + λ_i) mod p_i

* Position/state-dependent mixing
    * SHAKE128-derived λ_i(pos, state) prevents simple residue pattern leakage.
* Corridor certificate
    * Decoder can emit a certificate showing entropy levels, subset sizes, lane counts, corridor scores, and capacity margins.
* Byzantine demo mode
    * Includes consensus reconstruction over candidate subsets.
    * Detects corrupted shards in the prototype setting.

⸻

Important Status

This is a research prototype, not production cryptography.

It is suitable for:

* experimentation
* technical reports
* preprint development
* erasure coding research
* adaptive coding theory exploration
* HyperMorphic / HoloRAID-style systems research

It is not yet suitable for:

* production storage
* cryptographic secrecy claims
* replacing Reed–Solomon
* security-critical deployment
* formal Byzantine fault tolerance claims

⸻

Primitive Definition

An HCC system is parameterized as:

HCC(n, D, W, E_levels, k_low, k_high)

Where:

Parameter	Meaning
n	total number of shards
D	number of representation lanes / geometry axes
W	causal entropy window size
E_levels	entropy quantization levels
k_low	minimum shard count in low-complexity states
k_high	minimum shard count in high-complexity states

At each chunk position:

H(pos)     = entropy of previous W decoded chunks
e(pos)     = quantized entropy state
p_i(e)     = adaptive prime modulus for shard i
b_i(e)     = SafeGear base with gcd(b_i, p_i) = 1
λ_i(pos,e) = SHAKE128-derived position/state mixing term

Encoding:

r_i(pos) = (b_i(e) · C(pos) + λ_i(pos,e)) mod p_i(e)

Decoding:

c_i(pos) = (r_i(pos) - λ_i(pos,e)) · b_i(e)^(-1) mod p_i(e)
C(pos) = CRT({c_i mod p_i(e)} for i in selected corridor)

⸻

Corridor Predicate

A surviving shard subset S is accepted only if it satisfies:

product_capacity(S, e) > chunk_space
lane_count(S)          ≥ required_lanes(e)
|S|                    ≥ required_shards(e)
corridor_score(S, e)   ≥ required_score(e)

In symbolic form:

C(S, H_t) = 1

iff:

|S| ≥ k(H_t)
and L(S) ≥ ℓ(H_t)
and ∏ p_i(H_t) > 256^B
and σ(S, H_t) ≥ τ(H_t)

Where:

* H_t is the causal state
* L(S) is lane coverage
* B is chunk size in bytes
* σ(S, H_t) is corridor score
* τ(H_t) is the required corridor score

⸻

Demonstrated Novel Behaviour

The prototype demonstrates a key non-equivalence:

good_ids = [0, 1, 2, 3, 4, 5]      # 6 shards, all 4 lanes
bad_ids  = [0, 1, 4, 5, 8, 9]      # 6 shards, only lanes 0/1

Both subsets contain 6 shards.

But:

good subset → valid corridor → recovery succeeds
bad subset  → invalid corridor → recovery rejected

So HCC is not merely k-of-n.

It is:

recovery by invariant corridor span.

⸻

Example Test Output

RESULTS: 20/20 passed
ALL PASS — HCC primitive validated

Key tests include:

* basic roundtrip recovery
* low-entropy 4-shard recovery
* high-entropy 6-shard full-span recovery
* high-entropy clustered-subset rejection
* same-count different-geometry demonstration
* corridor certificate generation
* shard HMAC tamper detection
* Byzantine consensus demo
* erasure stress recovery

⸻


⸻

Minimal Usage

from hcc import HyperMorphicCorridorCode
hcc = HyperMorphicCorridorCode(
    n=12,
    geometry_dim=4,
    W=8,
    E_levels=9,
    low_shards=4,
    high_shards=6,
)
data = b"HyperMorphic Corridor Codes"
shards = hcc.encode(data)
recovered = hcc.decode(shards)
assert recovered == data

Decode with a certificate:

recovered, certificate = hcc.decode_with_certificate(shards)
print(certificate["entropy_level_dist"])
print(certificate["subset_size_dist"])
print(certificate["lane_count_dist"])
print(certificate["min_corridor_score"])

⸻

Example Corridor Certificate

{
    "entropy_level_dist": {0: 1, 2: 1, 3: 1, 4: 2, 5: 200},
    "subset_size_dist": {9: 205},
    "lane_count_dist": {4: 205},
    "min_corridor_score": 0.929,
    "min_capacity_margin_bytes": 7.58388
}

The certificate records the reconstruction path and shows why the selected surviving shards formed an admissible corridor.

⸻

Comparison With Fixed Threshold Coding

Property	Fixed Threshold Code	HyperMorphic Corridor Code
Counts surviving shards	yes	yes
Requires fixed k	yes	no
Uses state-dependent threshold	no	yes
Uses representation lanes	no	yes
Distinguishes same-count subsets	no	yes
Emits corridor certificate	no	yes
Uses adaptive prime topology	no	yes
Uses content/state mixing	optional	yes
Accepts/rejects by invariant span	no	yes

⸻

What This Is Not Claiming

This prototype does not currently claim:

* to beat Reed–Solomon
* to be production-ready
* to be cryptographically secure
* to prove a new MDS code family
* to provide formal Byzantine security
* to be information-theoretically superior to classical erasure codes

The intended claim is narrower and more defensible:

HCC demonstrates a state-adaptive reconstruction criterion where equal-cardinality shard subsets may differ in admissibility depending on algebraic capacity and geometric invariant span.

⸻

Suggested Formal Claim

Corridor Correctness

For any chunk position t, if the selected surviving subset S_t satisfies the corridor predicate, the moduli are pairwise coprime, and their product exceeds the chunk space, then CRT reconstruction returns the original chunk exactly.

Cardinality Non-Equivalence

There exist subsets S1 and S2 such that:

|S1| = |S2|

but:

C(S1, H_t) = 1
C(S2, H_t) = 0

Therefore, HCC recovery cannot be reduced to a fixed scalar threshold alone.

⸻

Roadmap

Planned improvements:

* add Reed–Solomon and fixed CRT baselines
* add HTC baseline comparison
* add capacity-only and lane-only ablations
* add adversarial erasure experiments
* add clustered failure benchmarks
* add optimized CRT implementation
* add serialization format for shards
* add reproducible benchmark notebook
* add formal theorem/proof section
* add larger real-world file recovery tests
* add HoloRAID integration layer

⸻

Suggested Citation

@misc{hypermorphic_corridor_codes,
  title  = {HyperMorphic Corridor Codes: State-Adaptive Reconstruction by Algebraic-Geometric Invariant Span},
  author = {Gerrard, Shaun Paul},
  year   = {2026},
  note   = {Research prototype}
}

⸻

License

Choose a license before release.

Recommended options:

* MIT License for maximum openness
* Apache-2.0 if patent-related clarity matters
* AGPL-3.0 if network-use openness matters

⸻

One-Line Summary

HyperMorphic Corridor Codes replace fixed threshold recovery with state-adaptive reconstruction by algebraic capacity and geometric invariant span.
