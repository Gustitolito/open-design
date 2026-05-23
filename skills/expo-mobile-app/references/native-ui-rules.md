# Native UI rules for Expo app generation

Use this reference when generating Expo mobile app artifacts for Open Design.

## Structure

- Use `app/` routes for multi-screen apps.
- Always ensure the app has a route for `/`.
- Use `_layout.tsx` for route layouts.
- Keep reusable components, utilities, types, and sample data outside `app/`.
- Use kebab-case for file names except conventional Expo entry files.

## Components

Prefer React Native and Expo-compatible components:

- `View`
- `Text`
- `ScrollView`
- `Pressable`
- `TextInput`
- `FlatList`
- `SectionList`
- `Image` from `expo-image` when image behavior matters
- `Link` from `expo-router` for navigation

Avoid web-only primitives unless implementing an explicit Expo DOM component:

- `div`
- `span`
- `button`
- `input`
- CSS stylesheets for normal React Native layout
- direct `document` or `window` usage

## Layout

- Build mobile-first.
- Prefer flexbox and `useWindowDimensions`.
- Prefer `ScrollView` for screens with variable height.
- Use `contentContainerStyle` for scroll padding and gaps.
- Avoid hard-coded full-screen absolute layouts unless the design requires it.
- Keep bottom navigation and sticky controls safe-area aware.

## Styling

- Use inline style objects or `StyleSheet` in the first implementation.
- Use design-system colors and typography when available.
- Prefer padding over margin for local spacing.
- Prefer consistent gap values.
- Use tabular numbers for counters and metrics where useful.
- Keep animations small and purposeful.

## Expo Go first

Prefer packages that work in Expo Go. Do not require `npx expo run:ios`, `npx expo run:android`, or EAS Build unless the user's requested feature needs custom native code.

## Web preview compatibility

The generated app must also work in Expo Web preview. Avoid native-only dependencies in default prototypes. When a native-only feature is requested, provide a web-safe placeholder state so the design preview still renders.
