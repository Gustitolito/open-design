# Expo Web preview runtime expectations

Open Design previews Expo mobile app artifacts through a daemon-managed Expo Web dev server.

## Runtime shape

Expected flow:

```txt
expo-mobile-app skill
  -> generated Expo project files
  -> live artifact with preview.type = expo-web
  -> daemon starts Expo Web
  -> web app renders the local dev server in a preview iframe
  -> UI exposes URL/QR handoff for phone preview when available
```

## Generated project requirements

The generated project must run from its project root with:

```bash
npx expo start
npx expo start --web
```

For Open Design preview, the daemon will use a controlled command similar to:

```bash
npx expo start --web --port <PORT> --host lan
```

The generated artifact should not depend on arbitrary custom shell scripts to render the first preview.

## Device handoff

The runtime may expose:

- local web URL, such as `http://localhost:<PORT>`;
- LAN web URL, such as `http://192.168.x.x:<PORT>`;
- Expo Go URL, when the Expo CLI provides one.

The UI should prefer Expo Go URL for native device handoff when available and fall back to LAN web URL otherwise.

## Limitations

Expo Web preview is not identical to native iOS/Android rendering. It is the fast design preview path. Native behavior should be verified separately in Expo Go or a development build when needed.

Do not use the Android emulator as the default preview path. It is an optional future advanced runtime.
