# Expo Skills upstream reference

This Open Design skill is based on the official Expo skills repository:

- Upstream repository: `https://github.com/expo/skills`
- Upstream plugin: `plugins/expo`
- License: MIT

Relevant upstream skills:

- `building-native-ui` — primary source for native UI, routing, Expo Go, styling, and responsiveness guidance.
- `expo-tailwind-setup` — optional future reference for NativeWind/Tailwind support. Not used by default in this Open Design MVP path.
- `native-data-fetching` — future reference for app prototypes with API calls, caching, and offline behavior.
- `expo-dev-client` — future reference for custom development clients when Expo Go is insufficient.
- `expo-deployment` — future reference for App Store, Play Store, and web deployment flows.

## Adaptation policy

Do not copy the full Expo plugin into Open Design blindly. Keep the Open Design skill small and focused on generation constraints that matter for previewable app prototypes.

The first implementation should prefer:

- Expo Go-compatible packages;
- Expo Router when navigation is needed;
- React Native primitives instead of HTML;
- Expo Web compatibility;
- minimal dependency footprint;
- generated projects that run outside Open Design with `npx expo start`.

NativeWind/Tailwind, development builds, EAS Build, and store deployment should remain follow-up capabilities, not default behavior for the initial preview runtime.
