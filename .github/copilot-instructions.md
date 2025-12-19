# GitHub Copilot instructions — auth0-react

Purpose: short, actionable guidance to help an AI coding agent be productive in this repository.

## Quick commands (use these first)
- Install repo deps: `npm ci`
- Prepare example apps: `npm run install:examples` (installs cra/gatsby/next/users-api)
- Build (lint + bundle): `npm run build` (runs `eslint`, then `rollup` → `dist/`)
- Dev server: `npm start` (rollup watch / dev server at http://localhost:3000)
- Unit tests: `npm test` (Jest + coverage)
- Integration / e2e (examples + Cypress): `npm run test:integration` or `npm run test:cra|test:gatsby|test:nextjs` (uses `start-server-and-test` and `cypress`)
- Run Cypress UI: `npm run cypress:open` (or `npm run test:cra:watch` for CRA flow)
- Generate docs (typedoc): `npm run docs`

## Big-picture architecture (what to know quickly)
- This repo is an SDK for React SPAs that delegates OAuth/OIDC flows to `@auth0/auth0-spa-js`.
- Public API surface is exported from `src/index.tsx` — **do not change public exports without updating docs and types**.
- Core pieces:
  - `src/auth0-provider.tsx`: provider lifecycle, `Auth0Client` instantiation, login/logout/token flows, redirect handling.
  - `src/use-auth0.tsx`, `src/with-auth0.tsx`, `src/with-authentication-required.tsx`: ergonomics for consumers.
  - `src/reducer.tsx` / `src/auth-state.tsx`: internal state transitions (e.g., `INITIALISED`, `GET_ACCESS_TOKEN_COMPLETE`).
  - `src/utils.tsx`: small helpers & deprecation mappings (e.g., `redirectUri` → `authorizationParams.redirect_uri`).
- Version is injected at build time (`__VERSION__`) via `rollup.config.mjs` — ensure this remains intact for release builds.

## Testing & mocking patterns (follow these)
- Tests use Jest + `@testing-library/react` + `ts-jest` (see `jest.config.js` and `jest.setup.js`).
- Auth client is mocked in `__mocks__/@auth0/auth0-spa-js.tsx`. Prefer mocking `Auth0Client` methods (e.g., `getUser`, `getTokenSilently`) instead of invoking network calls.
- Use `createWrapper` from `__tests__/helpers.tsx` to wrap components with `Auth0Provider` in unit tests.
- For adding behavior that affects state or lifecycle, add unit tests in `__tests__/*` and an integration test in the relevant example if needed.

## Project conventions & gotchas
- TypeScript is strict (`tsconfig.json`) — add precise types for public surfaces and tests.
- Linting: `npm run lint` (eslint + TypeScript parser) — ensure no new ESLint/TS errors.
- Backwards-compatibility is important: e.g., token exchange flow deliberately dispatches `GET_ACCESS_TOKEN_COMPLETE` (see code comment in `auth0-provider.tsx`) — changing these semantics needs tests and docs updates.
- Deprecations are handled explicitly (see `deprecateRedirectUri` in `src/utils.tsx`) — prefer issuing warnings over breaking changes.
- Examples live in `/examples`. Integration scripts assume `users-api` runs on port **3001** and example apps on **3000**.

## When you change public behavior
1. Add/modify unit tests and integration example tests if user-visible behavior changes.
2. Run `npm run lint`, `npm test`, `npm run build` locally.
3. Update `EXAMPLES.md` and `README.md` or add a short docs fragment and run `npm run docs` to regenerate typedoc (docs are under `docs/`).
4. Ensure the API export in `src/index.tsx` is updated and types are exported from built `dist`.

## Useful files to inspect when investigating bugs
- `src/auth0-provider.tsx` — initialization & redirect flows
- `src/use-auth0.tsx` — hook behavior
- `src/utils.tsx` — hasAuthParams, deprecations, error normalization
- `src/reducer.tsx` & `src/auth-state.tsx` — state machine & action names
- `__mocks__/@auth0/auth0-spa-js.tsx` — mock Auth0 behaviors used in tests
- `examples/*` & `package.json` scripts — how integration tests are wired (start-server-and-test + cypress)
- `rollup.config.mjs` — build outputs and `__VERSION__` replacement

## Quick PR checklist for changes
- Add or update unit tests (Jest)
- Run lint and build locally (`npm run lint` && `npm run build`)
- If public API changed, update docs and `EXAMPLES.md`, then run `npm run docs`
- If integration behavior changed, update the corresponding example and re-run `npm run test:integration`

---
If anything above is missing or unclear, tell me which areas you'd like expanded (examples, CI steps, or more file references).