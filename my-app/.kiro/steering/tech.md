# Tech Stack

## Core

- **Angular 22** (standalone components, signals, zoneless, AOT) — authored in TypeScript (`~6.0`, strict mode, `strictTemplates`).
- **Angular Native (`@ng-native/*` 0.24)** — renders Angular components as native views on React Native's Fabric renderer. Not React, no DOM.
- **React Native 0.88 / React 19.3** — the render path underneath; you rarely touch it directly.
- **Expo ~58** — the app shell and runtime.
- **Metro** — bundler, extended via `@ng-native/metro` to compile Angular (templates + per-component CSS) ahead of time.
- **Vitest 5** — tests run in Node against a fake native layer (`@ng-native/testing`), no simulator needed.

## Key conventions (see `AGENTS.md` for the full list)

- Native elements are **lowercase** (`<view>`, `<text>`, `<pressable>`, `<text-input>`, `<scroll-view>`, etc.) and must be imported from `@ng-native/components` into the component's `imports`. An uppercase name fails the build.
- No DOM: no `document`/`window`, no HTML elements (`<div>`, `<button>`, `<img>`), no `@angular/platform-browser`, no `@angular/animations`.
- State is signals and `computed()`. Nothing updates from a plain field changing outside a signal or event (zoneless).
- Events are native: `(press)` not `(click)`, `[(value)]`/`(changeText)` on `<text-input>`.
- Accessibility is props: `accessibilityRole`, `accessibilityLabel`, `[accessibilityState]` (web `role`/`aria-*` aliases work too).
- Styling: `[style]` takes a React Native style object (camelCase, points); a component's `styles` is real CSS compiled at build time. Default layout is flex, column direction.
- HTTP needs `provideNativeHttpClient()` from `@ng-native/platform/http`, not `provideHttpClient()`.
- Navigation uses `@angular/router` via `provideNativeRouter` from `@ng-native/router`; forms use Signal Forms (`@angular/forms/signals`).
- Platform-dependent services inject `OS`/`OS_VERSION` from `@ng-native/device` (never import `Platform` from `react-native`, tests can't load it).

## Common commands

```sh
npm start           # Metro; press i / a for a simulator, or scan the QR code with Expo Go
npm run ios         # iOS simulator
npm run android     # Android emulator
npm test            # Vitest in Node against a fake native layer (no simulator)
npm run typecheck   # Metro config check + ngc --noEmit
```

Run `npm test` and `npm run typecheck` after a change — both are fast. `npm run web` intentionally exits with an error; this app is iOS/Android only.
