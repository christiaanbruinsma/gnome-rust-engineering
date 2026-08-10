# RefCell borrow scopes must stay short around GTK callbacks

## Context / problem

Rust GTK code can panic at runtime when a mutable borrow lives too long and a signal or callback re-enters the same state.

## What happened?

A `RefCell` borrow was still active when a GTK callback path tried to access the same state again, which produced a borrow panic.

## Cause

The mutable borrow scope crossed a GTK callback boundary.

## Proven solution

Keep borrow scopes small. Drop mutable borrows before entering callbacks or re-entrant signal paths, and make ownership boundaries explicit.

## Evidence

The project handover records the `RefCell already borrowed` issue and the mitigation pattern around it.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not hold a `RefCell` borrow through code that may synchronously trigger GTK signal re-entry.
