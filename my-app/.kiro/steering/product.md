# Product

An Angular mobile app that renders **real native iOS and Android views** using [Angular Native](https://ng-native.com): Angular components running on React Native's Fabric renderer, inside an Expo app.

Key facts:
- Targets iOS and Android only. It is **not a web app** — there is no DOM, no JSX, no browser.
- Currently a starter scaffold (`src/app/app.ts` is a tap-counter demo) to build the real app on top of.
- Expo Go is enough for day-to-day development; a release build or an unbundled native module needs a development build (`npx expo run:ios` / `run:android`).
