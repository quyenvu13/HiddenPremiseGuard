# HiddenPremiseGuard v1.1

HiddenPremiseGuard is a GenLayer Intelligent Contract that detects whether a
proposal materially depends on a condition that its governing baseline neither
states nor reasonably entails.

It distinguishes new work from a new load-bearing premise. Adding an action,
detail, mechanism, scope, or implementation step is not automatically a hidden
premise. The proposal is blocked only when it would materially fail if an
unstated and unsupported condition were false.

## Semantic verdicts

- `NO_NEW_MATERIAL_PREMISE`
- `NEW_UNSUPPORTED_PREMISE`

Consensus is limited to this binary enum. Validators do not need to agree on a
free-form explanation or the name of the hidden premise.

## Deterministic consequence

The original baseline is immutable. Each baseline also stores an effective
baseline that initially equals the original text.

- `NO_NEW_MATERIAL_PREMISE`: the proposal is accepted, becomes active, and is
  appended to the effective baseline used for later evaluations.
- `NEW_UNSUPPORTED_PREMISE`: the proposal is blocked and the effective baseline
  remains unchanged.

Every later evaluation receives both the immutable original baseline and the
growing effective baseline. The original remains the controlling anchor, so a
sequence of accepted additions cannot erase, reinterpret, or weaken it.

All proposal attempts remain append-only. There is no cancel, withdraw,
deactivate, reset, baseline-edit, or proposal-edit method.

## Safety properties

- Model, transport, parsing, malformed-output, and non-convergence failures
  revert the transaction; the contract never manufactures a verdict.
- A failed semantic call writes no proposal, counter, effective baseline, or
  cache entry.
- Prompt fences are removed to a fixed point, with a punctuation fallback for
  deeply nested markers.
- Internal whitespace is normalized in the model-facing copy.
- Cache keys cover the exact original baseline, effective baseline, and
  proposal text seen by the model; they do not depend on mutable IDs.
- Cache hits do not consume the semantic-call budget.
- Each baseline is limited to 8 successful model evaluations and 100 recorded
  proposal attempts.
- Only the baseline owner may submit proposals.
- No global administrator, token logic, transfer logic, or clock is used.

## Contract interface

### Write methods

- `create_baseline(baseline_text: str)`
- `propose(baseline_id: int, proposal_text: str)`

### Read methods

- `get_config()`
- `get_baseline(baseline_id)`
- `get_proposal(proposal_id)`
- `get_attempt(baseline_id, attempt_id)`
- `get_attempts(baseline_id, from_id, count)`

`get_baseline` exposes the immutable `baseline_text`, growing
`effective_baseline_text`, `model_calls`, attempt count, active proposal count,
and blocked-premise count.

## Version identity

- Contract class: `HiddenPremiseGuard`
- Version: `1.1`
- Runtime: py-genlayer v0.2
- Network target: GenLayer StudioNet (`61999`)
- Deployment status: deployed and runtime-verified on StudioNet
- Contract address: `0x675D7214202F80aBf84EF8e0D1Ef0689768AA7d1`
- Explorer: <https://explorer-studio.genlayer.com/address/0x675D7214202F80aBf84EF8e0D1Ef0689768AA7d1>
- Source file: `contract/HiddenPremiseGuard.py`
- Source SHA-256: `d5c0eebfd8bd8f20c895da053517bd7f7ab17d0585e49866a66c55884d11c527`

This deployment uses the exact source identified by the SHA-256 above. Runtime
evidence, transaction hashes and observed post-state are recorded in
`TESTING.md`.

## Novelty boundary

This contract evaluates a proposal's prerequisite or presupposition: a
condition that must already hold for the proposal to work. That differs from a
strict-entailment question asking only whether one statement logically follows
from another. Portfolio-level comparison against any existing strict-entailment
project remains a required pre-submission review item.

## Validation status

Local syntax and source-invariant checks are documented in `TESTING.md`.
StudioNet verification covers both semantic verdicts, effective-baseline
growth and an exact cache replay. Two exploratory vectors that clarified the
boundary between new work and unsupported premises are also recorded there.

## Submission files

```text
contract/HiddenPremiseGuard.py
README.md
TESTING.md
```
