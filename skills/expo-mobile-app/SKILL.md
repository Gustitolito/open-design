---
name: expo-mobile-app
description: Create real Expo React Native mobile app prototypes with Expo Web preview and QR/device handoff. Use this when the user asks for an Expo app, React Native app, native mobile app, or mobile app that should become runnable code rather than an HTML phone mockup.
version: 0.1.0
license: MIT
triggers:
  - expo app
  - react native app
  - native mobile app
  - mobile app with expo
  - aplicativo expo
  - aplicativo react native
od:
  mode: prototype
  platform: mobile
  scenario: design
  preview:
    type: expo-web
    entry: package.json
    runtime:
      kind: dev-server
      qr: true
  design_system:
    requires: true
    sections: [color, typography, layout, components]
  outputs:
    primary: App.tsx
    secondary:
      - package.json
      - app.json
      - app/
      - components/
      - assets/
  capabilities_required:
    - file_write
---

# Expo Mobile App

Create a real Expo React Native project, not an HTML mobile mockup.

This skill is for runnable mobile app prototypes that should preview through Expo Web inside Open Design and open on a phone through Expo Go or a LAN URL when available.

## Required output shape

Create a complete Expo project in the artifact project root:

```txt
package.json
app.json
tsconfig.json
App.tsx
app/
  _layout.tsx
  index.tsx
components/
assets/
README.md
```

For simple prototypes, keep the project compact. For multi-screen prototypes, use Expo Router and create routes under `app/`.

## Core rules

1. Use React Native and Expo primitives, not HTML.
2. Do not output `div`, `span`, `button`, `input`, normal CSS files, `document`, `window`, or React DOM APIs unless the user explicitly asks for an Expo DOM component.
3. Prefer `View`, `Text`, `ScrollView`, `Pressable`, `TextInput`, `FlatList`, and Expo-compatible components.
4. Prefer Expo Router for navigation when the app has more than one screen.
5. Keep dependencies minimal and Expo-compatible.
6. The project must run with `npx expo start` and `npx expo start --web`.
7. Use inline React Native styles or `StyleSheet` for the first implementation. Do not introduce NativeWind/Tailwind unless explicitly requested.
8. Always create a route that resolves to `/`.
9. Keep reusable components outside `app/`, usually in `components/`.
10. Include realistic sample data when needed, but keep it local and deterministic.

## Package expectations

The generated `package.json` should include scripts like:

```json
{
  "scripts": {
    "start": "expo start",
    "web": "expo start --web"
  }
}
```

Use compatible Expo, React, React Native, React Native Web, and Expo Router versions. When exact versions are already present in the surrounding project or lockfile, follow those versions instead of inventing new ones.

## Native preview expectations

Open Design will use `od.preview.type: expo-web` to start an Expo Web dev server through the local daemon. The app should therefore work in web preview even if it is designed as a mobile app.

Avoid native-only modules in the default path. Use Expo Go-compatible packages first. Only require a custom development build when the user explicitly asks for features that need native code outside Expo Go.

## Styling guidance

- Build mobile-first layouts.
- Use safe-area-aware spacing where relevant.
- Prefer `ScrollView` for screens that may overflow.
- Use `useWindowDimensions` for responsive layout decisions.
- Prefer padding and gap over fragile absolute positioning.
- Use consistent spacing, type scale, and color tokens from the active design system.
- Add a small amount of interaction polish, but do not overbuild animations.

## References

Consult these local references as needed:

- `references/expo-skills-upstream.md` — how this skill relates to the official Expo Skills repository.
- `references/native-ui-rules.md` — condensed Expo/React Native UI rules for Open Design.
- `references/preview-runtime.md` — Open Design Expo Web preview expectations.
