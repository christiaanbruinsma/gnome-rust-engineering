# GtkListItemFactory teardown must clean up popovers

## Context / problem

A popover attached during item creation can survive longer than the GTK item row unless teardown explicitly removes it.

## What happened?

Data Inspector's per-cell context popovers needed explicit teardown to avoid GTK finalization warnings.

## Cause

The factory created a per-item popover but did not fully clean it up during item teardown.

## Proven solution

Clean up per-cell context popovers during `GtkColumnView` item teardown and detach them cleanly from their parent.

## Evidence

The Data Inspector changelog records the fix: per-cell context popovers were cleaned up during `GtkColumnView` item teardown to avoid GTK finalization warnings.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: no

## Pitfalls

Do not rely on implicit GTK object disposal when the row/item lifecycle is shorter than the attached popover's lifecycle.
