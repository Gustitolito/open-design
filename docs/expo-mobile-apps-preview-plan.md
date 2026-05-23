# Expo mobile apps preview plan

**Status:** planning
**Branch:** `Mobile-apps-expo`
**Scope:** first-class Expo/React Native mobile app generation with Expo Web preview and device handoff through QR/URL.

This document tracks the implementation plan for adding Expo mobile-app support to Open Design as a native preview runtime, not as an HTML-only mobile mockup.

## Design intent

Open Design currently treats most previews as static renderable artifacts. Expo apps are different: they are executable projects. The correct implementation is therefore a new runtime-backed preview path:

```txt
Expo skill -> Expo project artifact -> daemon runtime -> Expo Web dev server -> iframe preview + QR/device handoff
```

The implementation must preserve the existing HTML/JSX/Markdown preview path while adding a dedicated `expo-web` runtime for generated Expo projects.

## Upstream skill source

Use the official Expo skills repository as the upstream knowledge source:

- Repository: `https://github.com/expo/skills`
- Relevant plugin: `plugins/expo`
- Relevant skills:
  - `building-native-ui`
  - `expo-tailwind-setup`
  - `native-data-fetching`
  - `expo-dev-client`
  - `expo-deployment`

The first implementation should adapt only the minimum necessary guidance from the Expo skills, especially `building-native-ui`. Do not copy the full upstream plugin blindly. Open Design needs its own `expo-mobile-app` skill that declares OD-specific metadata, preview type, outputs, and runtime expectations.

## Non-goals for the first pass

- Do not implement Android Emulator preview.
- Do not require EAS Build.
- Do not require a custom development build.
- Do not make NativeWind/Tailwind the default path.
- Do not let generated artifacts execute arbitrary shell commands.
- Do not replace the existing `mobile-app` HTML/framed prototype skill.

## Completion criteria

The feature is complete when a user can ask Open Design to create a mobile app and receive a real Expo project that:

1. appears as an `expo-web` live artifact;
2. starts through the daemon;
3. renders in the Open Design preview panel through Expo Web;
4. exposes a local/LAN URL or QR for phone preview;
5. exports as a clean Expo project ZIP;
6. can run outside Open Design with `npx expo start`.

---

# To Do

## 0. Baseline and repository preparation

- [ ] Confirm the implementation branch is `Mobile-apps-expo`.
- [ ] Run the current app before changes.
- [ ] Record the exact current dev commands.
- [ ] Run baseline typecheck/tests.
- [ ] Confirm where built-in skills are loaded from.
- [ ] Confirm how Open Design parses `SKILL.md` frontmatter and `od:` extensions.

## 1. Architecture mapping

- [ ] Read `packages/contracts/src/api/live-artifacts.ts`.
- [ ] Read `apps/daemon/src/live-artifacts/schema.ts`.
- [ ] Read `apps/daemon/src/live-artifacts/store.ts`.
- [ ] Read `apps/daemon/src/live-artifact-routes.ts`.
- [ ] Read `apps/web/src/components/FileViewer.tsx`.
- [ ] Read `apps/web/src/providers/registry.ts`.
- [ ] Identify the exact preview switch/rendering point in `FileViewer.tsx`.
- [ ] Identify current live artifact create/update/preview routes.
- [ ] Identify current export ZIP flow.

## 2. Contracts and schema

- [ ] Add `expo-web` to `LiveArtifactPreviewType` in `packages/contracts/src/api/live-artifacts.ts`.
- [ ] Add `expo-web` to daemon preview validation in `apps/daemon/src/live-artifacts/schema.ts`.
- [ ] Add optional runtime metadata to `LiveArtifactPreview`.
- [ ] Define `LiveArtifactPreviewRuntime` with `kind`, `port`, `urlPath`, `qr`, and related fields.
- [ ] Keep existing `html`, `jsx`, and `markdown` artifacts backward-compatible.
- [ ] Add tests for valid and invalid preview metadata.

## 3. Expo project document model

- [ ] Add `expo_project_v1` as a new live artifact document format.
- [ ] Keep `html_template_v1` unchanged.
- [ ] Model Expo project artifacts as executable project directories, not rendered HTML templates.
- [ ] Validate artifact-relative project paths.
- [ ] Reject absolute paths and directory traversal.
- [ ] Store Expo project files under a dedicated `project/` directory.

Expected artifact layout:

```txt
.live-artifacts/
  la-example-app-xxxx/
    artifact.json
    project/
      package.json
      app.json
      App.tsx
      app/
      components/
      assets/
    runtime/
      preview-runtime.json
      expo.log
```

## 4. Daemon preview runtime system

Create a runtime subsystem, for example:

```txt
apps/daemon/src/preview-runtimes/
  types.ts
  index.ts
  expo-web.ts
  process-manager.ts
  port.ts
  log-buffer.ts
  local-network.ts
```

- [ ] Implement `PreviewRuntimeStatus`.
- [ ] Implement `ensureStarted`.
- [ ] Implement `restart`.
- [ ] Implement `stop`.
- [ ] Implement `getStatus`.
- [ ] Implement bounded runtime logs.
- [ ] Implement dynamic available-port selection.
- [ ] Prevent multiple runtime processes for the same artifact.
- [ ] Clean up child processes on daemon shutdown.

## 5. Expo Web runtime

- [ ] Run install only when needed.
- [ ] Prefer `pnpm` if available; otherwise fall back to `npm`.
- [ ] Start Expo from the artifact `project/` directory.
- [ ] Use a controlled command, not arbitrary artifact-provided shell.
- [ ] Start with:

```bash
npx expo start --web --port <PORT> --host lan
```

- [ ] Fall back to localhost host mode if LAN fails.
- [ ] Capture stdout/stderr.
- [ ] Parse local web URL.
- [ ] Parse LAN web URL when available.
- [ ] Parse Expo Go URL when available.
- [ ] Health-check `http://localhost:<PORT>` before reporting `ready`.
- [ ] Report `installing`, `starting`, `ready`, `stopped`, and `error` states.

## 6. Runtime HTTP API

Add runtime routes, preferably in a dedicated route file:

```txt
POST /api/live-artifacts/:artifactId/runtime/start
POST /api/live-artifacts/:artifactId/runtime/restart
POST /api/live-artifacts/:artifactId/runtime/stop
GET  /api/live-artifacts/:artifactId/runtime/status
GET  /api/live-artifacts/:artifactId/runtime/logs
```

- [ ] Require `projectId` query param.
- [ ] Validate artifact exists.
- [ ] Validate artifact preview type is `expo-web`.
- [ ] Validate document format is `expo_project_v1`.
- [ ] Return structured runtime status.
- [ ] Return user-readable errors.
- [ ] Do not modify the existing static `/preview` behavior for HTML/JSX/Markdown.

## 7. Web provider functions

Update `apps/web/src/providers/registry.ts`.

- [ ] Add `LiveArtifactRuntimeStatus` type.
- [ ] Add `startLiveArtifactRuntime`.
- [ ] Add `restartLiveArtifactRuntime`.
- [ ] Add `stopLiveArtifactRuntime`.
- [ ] Add `fetchLiveArtifactRuntimeStatus`.
- [ ] Add `fetchLiveArtifactRuntimeLogs`.

## 8. Expo preview UI

Create:

```txt
apps/web/src/components/ExpoPreviewPanel.tsx
```

- [ ] Auto-start runtime when opening an `expo-web` artifact.
- [ ] Poll runtime status while installing/starting.
- [ ] Render loading states.
- [ ] Render error states.
- [ ] Render runtime logs in a collapsible panel.
- [ ] Render Expo Web iframe once ready.
- [ ] Add controls: Restart, Stop, Open in browser, Copy URL.
- [ ] Reuse existing desktop/tablet/mobile viewport conventions.
- [ ] Use a mobile device frame for mobile viewport mode.
- [ ] Keep this inside the normal live artifact viewer flow.

For the Expo iframe, use a separate sandbox policy from static HTML previews if required by Metro/Expo Web:

```tsx
sandbox="allow-scripts allow-same-origin allow-forms allow-popups"
```

Document in code why this is scoped only to `expo-web`.

## 9. QR and device handoff

- [ ] Show LAN URL for phone preview.
- [ ] Add copy button for LAN URL.
- [ ] Add QR code rendering.
- [ ] Prefer Expo Go URL when available.
- [ ] Fall back to LAN web URL.
- [ ] Display clear same-network/firewall notes.
- [ ] Do not require a native emulator in this implementation.

## 10. Open Design Expo skill

Create a native OD skill, for example:

```txt
skills/expo-mobile-app/
  SKILL.md
  README.md
  references/
    native-ui.md
    route-structure.md
    expo-go-preview.md
    styling.md
    data-fetching.md
```

- [ ] Declare `od.preview.type: expo-web`.
- [ ] Declare primary output as Expo project files, not `index.html`.
- [ ] Use upstream Expo skills as references.
- [ ] Incorporate `building-native-ui` rules in condensed form.
- [ ] Require real React Native primitives.
- [ ] Require Expo Router structure when appropriate.
- [ ] Prohibit HTML-only output.
- [ ] Prohibit `div`, `span`, `button`, normal CSS files, `document`, and React DOM APIs unless explicitly using Expo DOM components.
- [ ] Prefer inline styles or React Native `StyleSheet` for MVP.
- [ ] Keep NativeWind/Tailwind as optional future path.

## 11. Expo project template expectations

Generated projects should contain a runnable baseline:

```txt
project/
  package.json
  app.json
  tsconfig.json
  App.tsx
  app/
    _layout.tsx
    index.tsx
  components/
  assets/
```

- [ ] Include `expo`.
- [ ] Include `react`.
- [ ] Include `react-native`.
- [ ] Include `react-native-web`.
- [ ] Include `expo-router` when using routes.
- [ ] Include scripts:

```json
{
  "scripts": {
    "start": "expo start",
    "web": "expo start --web"
  }
}
```

## 12. Export ZIP

- [ ] Add export mode for clean Expo project.
- [ ] Export only `project/` contents.
- [ ] Exclude `node_modules`.
- [ ] Exclude `.expo`.
- [ ] Exclude `runtime/`.
- [ ] Exclude logs and caches.
- [ ] Include README with run instructions.
- [ ] Verify exported ZIP runs outside Open Design.

## 13. Error handling

Handle and surface:

- [ ] Node/npm/pnpm unavailable.
- [ ] Expo install failure.
- [ ] Invalid `package.json`.
- [ ] Missing Expo dependency.
- [ ] Port conflict.
- [ ] Metro/Expo server failure.
- [ ] LAN URL unavailable.
- [ ] Firewall/network problem.
- [ ] Generated app uses unsupported native-only dependency.

## 14. Tests

### Contracts

- [ ] `expo-web` accepted.
- [ ] invalid preview types rejected.
- [ ] `expo_project_v1` accepted.
- [ ] invalid paths rejected.
- [ ] old artifacts still valid.

### Daemon

- [ ] runtime starts.
- [ ] runtime stops.
- [ ] runtime restarts.
- [ ] duplicate runtime is avoided.
- [ ] logs are bounded.
- [ ] no arbitrary shell execution from artifact metadata.

### Web

- [ ] `ExpoPreviewPanel` loads status.
- [ ] `ExpoPreviewPanel` renders iframe when ready.
- [ ] errors render properly.
- [ ] restart button calls runtime API.
- [ ] QR/URL renders when available.

### Manual E2E

Use this prompt:

```txt
Create a polished Expo mobile app prototype for a habit tracker with a home screen, stats screen, and profile screen. Use Expo Router and make it work in Expo Go.
```

Expected result:

- [ ] artifact created as `expo-web`;
- [ ] `project/package.json` exists;
- [ ] `project/app/_layout.tsx` exists;
- [ ] `project/app/index.tsx` exists;
- [ ] Expo Web opens in the preview panel;
- [ ] phone URL or QR appears;
- [ ] ZIP export runs outside Open Design.

## 15. Documentation

- [ ] Link this plan from `docs/spec.md`.
- [ ] Add final architecture notes to `docs/architecture.md` after implementation.
- [ ] Update `docs/skills-protocol.md` once `expo-web` becomes a supported preview type.
- [ ] Add user-facing docs for Expo preview limitations.
- [ ] Document difference between Expo Web, Expo Go, development builds, and native emulator.

## 16. Suggested PR sequence

1. Contracts and schema.
2. Expo skill and docs.
3. Daemon preview runtime.
4. Web provider functions and `ExpoPreviewPanel`.
5. QR/device handoff.
6. Export ZIP and final docs.

## Final acceptance checklist

- [ ] Typecheck passes.
- [ ] Tests pass.
- [ ] Existing HTML/JSX/Markdown previews still work.
- [ ] Existing `mobile-app` skill still works.
- [ ] New `expo-mobile-app` skill generates an actual Expo project.
- [ ] Expo Web preview works in Open Design.
- [ ] Device QR/URL is available.
- [ ] Exported project runs independently.
