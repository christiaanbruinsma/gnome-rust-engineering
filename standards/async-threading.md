# Async and threading

The GTK main loop must stay responsive.

Heavy work belongs off the UI thread or in explicit background tasks, and results should be passed back into the UI in a controlled way.

Avoid keeping borrow scopes or widget state active across work that can re-enter UI code. Treat `spawn_blocking` and similar boundaries as explicit contracts that may return nested errors or panic payloads.
