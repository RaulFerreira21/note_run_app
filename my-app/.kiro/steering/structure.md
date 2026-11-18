# Project Structure

The app lives in the `my-app/` directory.

```
my-app/
├── src/
│   ├── main.ts            # Entry point: registers platform components and mounts the root component via @ng-native/platform
│   └── app/
│       ├── app.ts         # Root component (selector: app-root) — the demo tap counter, build your app here
│       └── app.test.ts    # Example Vitest test alongside the component
├── assets/                # App icons and splash images
├── app.json               # Expo config (name, platforms, icons, router root, plugins)
├── metro.config.js        # Metro + withAngularNative (AOT compile, CSS, polyfills)
├── vitest.config.mts      # Vitest + ngNative() plugin
├── tsconfig.json          # TypeScript + Angular compiler options (strict, strictTemplates)
├── package.json           # Dependencies and npm scripts
├── AGENTS.md              # Authoritative framework rules for coding agents — read before changing code
└── CLAUDE.md              # Points to AGENTS.md
```

## Organization conventions

- **One component per folder** under `src/app/`, with its test (`*.test.ts`) alongside it. Write or update a test with each change — tests run in Node, so no simulator is needed.
- Components are **inline**: template and styles live in the `@Component` decorator (see `app.ts`). No separate `.html`/`.css` files.
- `src/main.ts` is the only mount point; it wires in device conditions, tokens, color processing, and asset resolution. Don't duplicate this setup elsewhere.
- Component hosts are flex items with no `flex` by default — a component that should fill its space sets `host: { style: 'flex: 1' }` or `:host { flex: 1; }`.
- Routing is configured to root at `src/app` (`app.json` → `extra.router.root`).
- `AGENTS.md` is the source of truth for framework-specific rules; extend it as the app grows.
