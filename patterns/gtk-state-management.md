# GTK state management

GTK state should remain small, explicit, and easy to reason about.

Prefer clear ownership boundaries over global mutable state. When shared state is needed, ensure it cannot be borrowed mutably across re-entrant UI paths.

A `RefCell` or similar shared mutable container should not be held across GTK callbacks unless the callback path has been proven safe.
