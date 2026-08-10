# Error handling

Errors should be surfaced with enough context to diagnose the failure boundary.

Preferred properties:

- the failure is visible;
- the message identifies the stage or boundary;
- the application does not panic for expected user input failures;
- background and blocking work propagate errors explicitly rather than collapsing them into a generic success/failure result.
