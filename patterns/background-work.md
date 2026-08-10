# Background work

Background work should keep the GTK main loop responsive.

Use asynchronous or off-thread work for heavy filesystem operations, scanning, exports, subprocesses, hashing, or other tasks that could block the UI.

Return results to the UI layer explicitly. Keep UI state, progress presentation, and result rendering separate from the expensive work itself.

Do not let background tasks hold borrow scopes or GUI state in a way that can re-enter the same state from a callback.
