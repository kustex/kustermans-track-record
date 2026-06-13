# Kustermans Dynamics Track-Record Attestations

This repository contains cryptographically chained attestations of portfolio
events — **trades** and **cash flows** (seed capital, deposits, withdrawals).
Each event is hashed (SHA-256) together with the previous attestation hash,
forming a tamper-evident, append-only chain. Recording cash flows alongside
trades makes the track record independently *verifiable*: the capital base and
every movement across the portfolio boundary are on the chain, so returns can
be reconstructed and nothing can be moved off-record.

## Event kinds

- `trade` — a buy/sell, with an optional real-time market-price snapshot.
- `cashflow` — a `seed`, `deposit`, or `withdrawal` (amount + currency + date).
- `amend` — corrects an earlier cash-flow attestation; carries the new values
  and references the original `sequence_number`. The chain is never mutated.
- `void` — records the deletion of an earlier cash-flow attestation.

## How it works

1. An event occurs in the Kustermans Dynamics application.
2. Event data + the previous attestation hash are combined into a canonical
   (sorted-keys) JSON payload and hashed with SHA-256.
3. The attestation is committed to this repository with a timestamp.

## Verification

Recompute every hash from the stored JSON files and check that each
`previous_hash` matches the preceding attestation's `attestation_hash`. The
first attestation's `previous_hash` is the genesis hash:
`SHA-256("KUSTERMANS-ATTESTATION-GENESIS-v2")`. Each file contains exactly the
fields that enter its hash, so it can be verified standalone.

## What this proves

- Each event was recorded at the time shown in the git commit timestamp
  (independently verifiable via GitHub's API).
- The hash chain is append-only: inserting, removing, or modifying any
  attestation breaks the chain.
- Corrections are transparent: edits and deletions appear as `amend`/`void`
  events rather than silent rewrites.
