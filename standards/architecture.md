# Architecture

The suite's architectural baseline is local-first, native GNOME, and evidence-driven.

Principles:

- keep application identity, build identity, and packaging identity in sync;
- separate UI, background work, and validation concerns;
- keep project-specific features separate from reusable platform rules;
- use the platform's native widgets and validation tools before introducing custom abstractions;
- promote a rule to standard only after it has been proven reusable.
