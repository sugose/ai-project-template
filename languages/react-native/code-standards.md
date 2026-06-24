## React Native / Expo Code Standards

- **Expo SDK 54** — target physical device testing via Expo Go; no EAS build required
- **TypeScript strict mode** — no `any`, no implicit returns
- **Biome** for linting and formatting — `npx biome ci .` must be clean before pushing
- **jest-expo** preset for testing — unit tests only; no device or emulator tests in CI
- **`@testing-library/react-native`** for component tests
- **StyleSheet** for all styles — no inline style objects
- **No hardcoded values** — API keys and environment-specific config via `.env` (gitignored); access via `expo-constants` or `process.env`
- **Platform guards** — use `Platform.OS` or `.native.tsx` / `.tsx` file extensions for platform-specific behaviour
- **Throttle-aware API client** — inspect response headers (`X-RequestCounter-Reset`, `X-Requests-Available`) and back off automatically; never hammer the API

### Project structure

```
src/
  app/              # Expo Router file-based routes (_layout.tsx, index.tsx, etc.)
  components/       # Shared UI components
  services/         # API clients and data fetching
  hooks/            # Custom React hooks
  types/            # TypeScript type definitions
__tests__/          # Jest tests (mirrors src/ structure)
```

### Running the project

```bash
# Install dependencies
npm ci

# Start Expo dev server (scan QR code with Expo Go on Android)
npx expo start

# Lint and format check
npx biome ci .

# Format (auto-fix)
npx biome format --write .

# TypeScript check
npx tsc --noEmit

# Run tests
npx jest
```

### Environment variables

All secrets and environment-specific values live in `.env` at the project root. This file is gitignored and never committed.

```
# .env
EXPO_PUBLIC_API_KEY=your_key_here
EXPO_PUBLIC_API_BASE_URL=https://api.football-data.org/v4
```

Variables prefixed with `EXPO_PUBLIC_` are accessible in app code via `process.env.EXPO_PUBLIC_*`. Never use unprefixed variables for values needed at runtime in the app bundle.

### Throttling

Always inspect API response headers and respect rate limits. Log a warning when requests are running low. Back off and retry after the reset window rather than failing hard.
