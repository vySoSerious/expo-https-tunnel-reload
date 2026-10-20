# Metro reload never reaches iOS through an https URL without a port

A blank `create-expo-app` (SDK 58) plus `expo-dev-client`, nothing else.

When the dev server URL is `https://<host>` with no port, the iOS command socket (`/message?role=ios`) connects to `<host>:8081` instead of `<host>:443`. So `r` in the terminal does nothing.

## Reproduce

Needs Xcode and [`cloudflared`](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/).

```sh
npm ci
npx expo run:ios --no-bundler                     # build and install the dev build
cloudflared tunnel --url http://localhost:8081    # in a second terminal; prints https://<id>.trycloudflare.com
EXPO_PACKAGER_PROXY_URL=https://<id>.trycloudflare.com npx expo start --dev-client
```

Open the `› Metro:` link it prints (`xcrun simctl openurl booted '<link>'`, then tap Open). Wait for `iOS Bundled`, then press `r`.

- Expected: the app reloads, and Metro prints a new `iOS Bundled` line.
- Actual: Metro prints `› Reloading apps` and nothing else. The app doesn't reload.

The app's log shows it dialing port 8081 on the tunnel host:

```sh
xcrun simctl spawn booted log stream --level debug --predicate 'process == "expohttpstunnelreload"' | grep 'tcp_connection_create_with_endpoint'
```

## Control

Stop Metro, run `npx expo start --dev-client` (no tunnel, no proxy URL), open its `› Metro:` link the same way, and press `r`. The app reloads.
