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

This PR itself triggers a post-merge admission run on `master`
(`5e676378660ea76f635ed3db9a7398dff251272b`). That run id is the control base
for `ax-evidence-negative-control.yml`; results are appended here after the run.

## Non-claims

- `verify-export` reuses the same engine's tier authority. It re-derives from
  exported bytes and a pinned verifier; it is not an independent semantic
  re-implementation.
- GitHub attestation establishes artifact provenance, not semantic truth.
- Digest equality is asserted between two legs of one admission run on one
  subject; it is not a claim of digest equality across different subject commits.
