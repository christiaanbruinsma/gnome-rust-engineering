# A test fixture must reach the branch it claims to test

## Context / problem

A regression test can be named after one failure mode while its fixture accidentally triggers an earlier guard. In that case, the test may fail or pass without ever exercising the intended branch.

## What happened?

Delivery Hub v0.9.0 contained a verifier test named for changed payload checksum behavior. The fixture replaced the original payload with different content, but the replacement also had a different file size.

The verifier correctly checks file size before SHA-256 equality. Because the size changed first, verification stopped at the size mismatch and the checksum mismatch branch was never reached.

The test therefore did not isolate the behavior its name and assertions were intended to validate.

## Cause

The fixture changed more than one relevant property at the same time:

- file contents changed;
- file length changed;
- checksum changed as a consequence.

An earlier validation guard consumed the case before the intended checksum branch.

## Proven solution

Construct the fixture so that all earlier guards deliberately remain satisfied and only the property under test changes.

For the Delivery Hub verifier test, the replacement payload was changed to content with the same byte length as the original fixture. That preserved the file-size check while changing the SHA-256 digest.

The targeted checksum test then passed, followed by the complete suite passing 46/46 tests.

## Evidence

- Initial full `cargo test`: 45 passed, 1 failed.
- Failing test: `services::verify::tests::changed_payload_is_exposed`.
- Source inspection showed file-size validation precedes checksum validation.
- Original mutation changed the payload length.
- Same-length changed payload allowed the intended checksum branch to execute.
- Targeted test: PASS.
- Full suite after correction: 46/46 PASS.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: reusable testing rule

## Pitfalls

- A test name does not prove which code path executed.
- An assertion about a downstream failure mode does not prove earlier guards were bypassed correctly.
- Avoid fixtures that mutate several validated properties unless the test intentionally covers their precedence.
- When validation is ordered, design fixtures with that ordering in mind.

## Provenance

- Originating project: Delivery Hub v0.9.0
- Gate or phase: release `cargo test`
- Evidence type: failing executed test, verifier source inspection, corrected fixture, targeted and full-suite rerun
- Validation result: targeted PASS; full suite 46/46 PASS
