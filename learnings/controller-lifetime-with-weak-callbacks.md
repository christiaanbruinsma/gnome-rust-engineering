# Controller lifetime must outlive Weak-only callbacks

## Context / problem

A GTK callback can keep working only if the object graph behind it still has a live owner. A `Weak<Self>` on its own is not enough when no strong owner remains to drive the callback chain.

## What happened?

In Signature Designer, a controller was refactored so that only a weak reference remained reachable by the callback path. The callback then had nothing live to operate on and appeared to do nothing.

## Cause

The controller lifetime had been shortened too far. The callback path no longer had a valid owning reference in the place where GTK needed it.

## Proven solution

Keep the actual owning reference alive for as long as GTK callbacks need it, and use weak references only as a non-owning escape hatch to avoid cycles.

## Evidence

The Signature Designer handover recorded the controller-lifetime bug and its fix boundary, including the shift away from a dead callback path.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: no

## Pitfalls

Do not replace ownership with `Weak` just to “make the cycle go away” unless the live owner still exists elsewhere.
