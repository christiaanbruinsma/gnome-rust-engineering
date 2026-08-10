# GTK4

GTK4 standards in this repository favor native widgets, semantic icon names, and explicit teardown/lifecycle handling.

The UI should remain calm, functional, and easy to audit. Avoid custom widget behavior unless the native widget set cannot express the required interaction.

Common suite conventions include a left file/navigation area, a centered working area, and a right inspector area where appropriate.