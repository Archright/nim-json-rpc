# Certificate-evidence acceptance — 2026-11-20 (Axiom #188)

Status: operational acceptance record. This file is evidence, not a contract.

## What is being accepted

The admission path of `Archright/nim-json-rpc` consumes an engine-produced
certificate evidence package instead of trusting an in-run manifest cache:

- producer: `Archright/axiom-cli` v0.6.6, `ax cert export` (certificate bodies +
  canonical certificate-set manifest + `certificateSetDigest`)
- provenance: `actions/attest@v4` with predicate type
  `https://github.com/Archright/Axiom/attestation/certificate-set/v1`
- consumer: `certificate-evidence` job in
  `.github/workflows/nim-json-rpc-pr-admission.yml`
- verifier: `ax cert verify-export --closure --project` on both legs, re-deriving
  the tier from the exported bodies with the existing engine authority

The existing signature/integrity, proposition-identity, mutation-contrast and
aggregate verdict guards are unchanged; this layer is added, not substituted.

## First live consumer run (PR #4)

- PR: `Archright/nim-json-rpc#4` — merged at `2026-11-20T06:51:25Z`
- merge commit: `5e676378660ea76f635ed3db9a7398dff251272b`
- workflow run: `44689066001` (head `6b133cb`, conclusion `success`)
- `certificate-evidence` job: 55s, conclusion `success`

`certificate-evidence-report.json` from that run:

```json
{
  "schema": "certificate-evidence-report/1",
  "verdict": "PASS",
  "evidence_tier": "verified",
  "certificate_set_digest": "0957d64f5044ec4140b73b990f22f46e3af24c910b0962ab19985988ebd9c863",
  "leg_digests": {
    "A": "0957d64f5044ec4140b73b990f22f46e3af24c910b0962ab19985988ebd9c863",
    "B": "0957d64f5044ec4140b73b990f22f46e3af24c910b0962ab19985988ebd9c863"
  },
  "artifact_digests": {
    "A": "sha256:189dacf7923db8533809d3d3fd54c8d546980df90f1f8896569ad5a69c64ddde",
    "B": "sha256:daa26d437c7a6aa64c2f7affc1540d0bed8df1f85541bbfbe00f48f51ca49f7c"
  },
  "guards": {
    "CERTIFICATE_EVIDENCE": "PASS",
    "CERTIFICATE_INTEGRITY": "PASS",
    "ARTIFACT_PROVENANCE": "PASS",
    "LEG_DIGEST_EQUALITY": "PASS"
  },
  "failures": []
}
```

Both legs executed independently (separate jobs) and produced the same
`certificateSetDigest`. Artifact archives were bound to the GitHub artifact API
digest before extraction, and each archive's attestation was verified with the
signer pinned to `Archright/axiom-cli/.github/workflows/ax-verify.yml`.

## Canonical-master run and negative controls

This PR triggers an admission run whose `pull_request` subject is the merge of
this branch into `master` (`5e676378660ea76f635ed3db9a7398dff251272b`). That run
is the control base for `ax-evidence-negative-control.yml`.

- admission run: `44690942294` (head `c3eb84fa67b975ffeb50f5ae31845f6aefd705fc`,
  conclusion `success`, `certificate-evidence` job `success`)
- leg digests (equal): `f326af6710e9d6591bf74d34d0748ea202da17209781dae657055d3423020ca5`
- artifact digests: A `sha256:b10c713d9c7070fd0fae44ac615635be2e02e69d17e17e8d510a2f9316d5a78b`,
  B `sha256:9d479375bb312e4b0aafe5f433cac3e9bf4c55581c28985acb7f22e5f7a809f5`
- all four evidence guards PASS: `CERTIFICATE_EVIDENCE`, `CERTIFICATE_INTEGRITY`,
  `ARTIFACT_PROVENANCE`, `LEG_DIGEST_EQUALITY`

Negative-control run `44695314036` (`ax-evidence-negative-control.yml`, base
`44690942294`, base digest `f326af6710e9d6591bf74d34d0748ea202da17209781dae657055d3423020ca5`):
verdict `PASS`, 4/4 controls rejected with the exact contract reason code.

| mutation | expected | observed | exit |
|---|---|---|---|
| body-removal | `CERT_BODY_MISSING` | `CERT_BODY_MISSING` | 1 |
| status-tamper | `CERT_STATUS_MISMATCH` | `CERT_STATUS_MISMATCH` | 1 |
| digest-tamper | `CERT_SET_DIGEST_MISMATCH` | `CERT_SET_DIGEST_MISMATCH` | 1 |
| observation-loss | `AXIOM_OBSERVATION_MISSING` | `AXIOM_OBSERVATION_MISSING` | 1 |

Each copy breaks exactly one property while keeping the other integrity layers
consistent, so the reason codes prove the intended guard fired instead of a
generic digest failure.

## Non-claims

- `verify-export` reuses the same engine's tier authority. It re-derives from
  exported bytes and a pinned verifier; it is not an independent semantic
  re-implementation.
- GitHub attestation establishes artifact provenance, not semantic truth.
- Digest equality is asserted between two legs of one admission run on one
  subject; it is not a claim of digest equality across different subject commits.
