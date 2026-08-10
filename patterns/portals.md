# Portals

Use portals and platform-native access boundaries when the application needs sandbox-safe access to files or host features.

Do not assume a desktop app can directly access the host filesystem or host services simply because it works on the developer machine.

Validate portal and permission behavior in the actual Flatpak/runtime context.