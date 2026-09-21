# HiddenPremiseGuard v1.1 — Testing

## Verified deployment

- Network: GenLayer StudioNet (`61999`)
- Contract: `0x675D7214202F80aBf84EF8e0D1Ef0689768AA7d1`
- Explorer: <https://explorer-studio.genlayer.com/address/0x675D7214202F80aBf84EF8e0D1Ef0689768AA7d1>
- Contract source SHA-256:

```text
d5c0eebfd8bd8f20c895da053517bd7f7ab17d0585e49866a66c55884d11c527
```

The source hash above is unchanged from the locally frozen v1.1 source.

## Verification summary

The deployed contract demonstrated all essential semantic paths:

- an unsupported premise was blocked;
- a new action without a hidden prerequisite was accepted;
- accepted text extended the effective baseline without changing the original;
- an explicit unsupported capacity assumption was blocked;
- replaying the identical blocked proposal used cache;
- the cache replay increased the attempt count without increasing model calls.

TX1 through TX6 were observed as `FINALIZED / SUCCESS`. TX7 was observed as
`ACCEPTED / SUCCESS`; its post-state was immediately readable and confirmed the
cache invariants below. The explorer link above is the authoritative live source
for later finality status.

Individual validators may display `execution cancelled after quorum`. This is
expected after consensus has already been reached and does not change a
successful transaction result.

## Baseline

Baseline ID `1` was created with this exact text:

```text
Grant applications are reviewed by two staff members.
```

### TX1 — create baseline — PASS

```text
method: create_baseline
baseline_id: 1
attempt_count: 0
model_calls: 0
active_proposal_count: 0
premise_blocks: 0
transaction: 0x9245026c1c8d49c52f7772ff83dfbc4b2aeaa20a34b4452f338f759750a72e89
status: FINALIZED / SUCCESS
```

## Semantic runtime vectors

### TX2 / K1 — unsupported timing premise — PASS

```text
The second reviewer signs off remotely within the same business day.
```

Observed:

```text
attempt_id: 1
proposal_id: 1
verdict: NEW_UNSUPPORTED_PREMISE
accepted: false
active: false
used_cache: false
attempt_count: 1
model_calls: 1
active_proposal_count: 0
premise_blocks: 1
transaction: 0xaeebb05210cc3016875b3549c8bb6dc7278d940786e957021ae8a563cea2ad74
status: FINALIZED / SUCCESS
```

### TX3 / exploratory boundary — unsupported asserted fact

```text
Because both reviewers already record their decision in writing, the file will also carry a one-line summary of each decision.
```

Observed verdict: `NEW_UNSUPPORTED_PREMISE`.

The phrase “already record their decision in writing” asserts a condition not
stated or guaranteed by the baseline. The contract correctly treated that
assertion as unsupported. This exploratory vector was therefore replaced by
the direct-action vector in TX4.

```text
attempt_id: 2
used_cache: false
attempt_count: 2
model_calls: 2
active_proposal_count: 0
premise_blocks: 2
transaction: 0xdc787d1ee3bacc24fa6868f2289a5c7658347f4116c7c46523b3a02fb404a21b
status: FINALIZED / SUCCESS
```

### TX4 / K2 — direct new action — PASS

```text
Add a one-line review summary to each grant application.
```

Observed:

```text
attempt_id: 3
proposal_id: 3
verdict: NO_NEW_MATERIAL_PREMISE
accepted: true
active: true
used_cache: false
attempt_count: 3
model_calls: 3
active_proposal_count: 1
premise_blocks: 2
transaction: 0xa2d4252ca0edc138c480ce8630e90a6da8cd8f619bb1c8515720dacc60c14793
status: FINALIZED / SUCCESS
```

The exact K2 text was appended to `effective_baseline_text`. The immutable
`baseline_text` remained unchanged.

### TX5 / exploratory boundary — operational commitment

```text
Applications are queued and cleared in a single weekly batch.
```

Observed verdict: `NO_NEW_MATERIAL_PREMISE`.

This sentence is a direct operational commitment. It does not itself assert a
separate prerequisite that must already be true. The result is consistent with
the contract rule that new work is not automatically a hidden premise. The
explicit-capacity vector in TX6 tests the intended unsupported dependency.

```text
attempt_id: 4
used_cache: false
attempt_count: 4
model_calls: 4
active_proposal_count: 2
premise_blocks: 2
transaction: 0x63c7b334b7d7861e2dd87445946e00832b956e34107ae9b20d4facd5f8568e40
status: FINALIZED / SUCCESS
```

### TX6 / K3 — explicit unsupported capacity premise — PASS

```text
Applications will be cleared in one weekly batch on the assumption that the two staff members can process the entire week's volume in one sitting.
```

Observed:

```text
attempt_id: 5
proposal_id: 5
verdict: NEW_UNSUPPORTED_PREMISE
accepted: false
active: false
used_cache: false
attempt_count: 5
model_calls: 5
active_proposal_count: 2
premise_blocks: 3
transaction: 0xd142baf250c050f95df7d42c72ffb748cdcb82c5b0e92391beb6e6b04883887b
status: FINALIZED / SUCCESS
```

The effective baseline did not change after this blocked attempt.

### TX7 / cache replay — PASS

TX6 was submitted again with identical baseline ID, proposal bytes, original
baseline and effective baseline.

Observed:

```text
attempt_id: 6
proposal_id: 6
verdict: NEW_UNSUPPORTED_PREMISE
accepted: false
active: false
used_cache: true
attempt_count: 6
model_calls: 5
active_proposal_count: 2
premise_blocks: 4
transaction: 0x4cb37409ba53a0b842b41e5a308eec3c3b0fce0b84790fc47648b493723da58e
status at evidence capture: ACCEPTED / SUCCESS
```

The decisive cache invariant passed: `attempt_count` increased from 5 to 6,
while `model_calls` remained 5.

## Final observed baseline state

```text
baseline_id: 1
attempt_count: 6
model_calls: 5
active_proposal_count: 2
premise_blocks: 4
```

The effective baseline contains only the two accepted proposals from TX4 and
TX5. Blocked attempts did not modify it.

## Local verification — 2026-09-21

The following checks passed locally without invoking a GenLayer model:

- Python syntax compilation.
- `Depends` remains the first line.
- No storage `TreeMap` was initialized inside `__init__`.
- Exactly one `run_nondet_unsafe` call exists.
- Unauthorized and invalid-baseline calls revert before semantic work.
- A simulated semantic failure leaves proposal and baseline state unchanged.
- Fixed-point fence removal, whitespace normalization and case normalization.
- Prompt/runtime-vector token overlap remained below the required limit.

Negative authorization and invalid-ID cases were verified locally and were not
repeated as on-chain writes.

## Reproduction

Read configuration and state with:

```text
get_config()
get_baseline(1)
get_attempt(1, 1)
get_attempt(1, 3)
get_attempt(1, 5)
get_attempt(1, 6)
```

Expected configuration invariants:

```text
name = HiddenPremiseGuard
version = 1.1
max_model_calls_per_baseline = 8
effective_baseline_grows = true
original_baseline_immutable = true
```
