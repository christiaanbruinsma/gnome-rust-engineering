# GTK signals

GTK signals and callbacks should be written with explicit ownership and borrow boundaries.

Practical rules:

- keep mutable borrow scopes short;
- avoid holding a borrow across a callback that can re-enter the same state;
- clone owned values deliberately when a callback needs its own copy;
- use weak references only when a live owner still exists elsewhere;
- clean up attached GTK objects during the relevant item or widget teardown path.
