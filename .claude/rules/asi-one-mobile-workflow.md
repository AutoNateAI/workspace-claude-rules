# ASI:One Mobile Development Workflow

Guidelines for pulling code, building, and running the ASI:One mobile app correctly.

## When to Use

Attach this rule when you say:
- "pull latest mobile code"
- "update asi-one-native"
- "build the mobile app"
- "fix react hook errors"
- "run the ios simulator"

---

## Critical: React Version Conflict

The monorepo has a React version conflict:

| Branch | Lockfile Optimized For | React Version |
|--------|------------------------|---------------|
| `staging` | Web apps (Next.js) | React 19.x |
| `asi-one-mobile-staging` | Mobile apps (RN) | React 18.x |

**Installing from `staging` will break React Native with hook errors.**

---

## Pulling Latest Code

### Standard Update (Most Common)

When you need to get the latest code from staging:

```bash
# Step 1: Go to clients repo
cd /Users/nathan.baker/code/fetch_workspace/clients

# Step 2: Stash any local changes
git stash

# Step 3: Update staging first (get latest code)
git checkout staging
git pull origin staging

# Step 4: Switch to mobile-staging and install (get correct React)
git checkout asi-one-mobile-staging
git pull origin asi-one-mobile-staging
pnpm i && pnpm build

# Step 5: Switch back to staging (KEEP the node_modules!)
git checkout staging

# Step 6: Regenerate API client from latest OpenAPI spec
pnpm run orval

# Step 7: Build chat package against staging code (must run after switching back)
pnpm build:chat

# Step 8: Restore local changes if any
git stash pop

# Step 9: Start dev server (do NOT run pnpm i!)
cd apps/asi-one-native
pnpm dev
```

### Feature Branch Workflow

When creating a new feature branch:

```bash
# Step 1: Create feature branch from staging
cd /Users/nathan.baker/code/fetch_workspace/clients
git checkout staging
git pull origin staging
git checkout -b your-feature-branch

# Step 2: Install from mobile-staging
git checkout asi-one-mobile-staging
git pull origin asi-one-mobile-staging
pnpm i && pnpm build

# Step 3: Switch back to feature branch
git checkout your-feature-branch

# Step 4: Regenerate API client from latest OpenAPI spec
pnpm run orval

# Step 5: Build chat package against feature branch code (must run after switching back)
pnpm build:chat

# Step 6: Start dev server (do NOT run pnpm i!)
cd apps/asi-one-native
pnpm dev
```

---

## Building and Running

### Start Dev Server

```bash
cd /Users/nathan.baker/code/fetch_workspace/clients/apps/asi-one-native
pnpm dev
```

Then press `i` for iOS simulator or `a` for Android emulator.

### Build Development Client (if needed)

If you see "No development build installed":

```bash
cd /Users/nathan.baker/code/fetch_workspace/clients/apps/asi-one-native
npx expo run:ios --device "iPhone 17 Pro"
```

Or use EAS for a dev build:

```bash
eas build --profile development-simulator --platform ios
```

---

## Troubleshooting

### React Hook Errors ("Invalid hook call")

This means you have multiple copies of React. Fix:

```bash
# Clean and reinstall from mobile-staging
cd /Users/nathan.baker/code/fetch_workspace/clients
rm -rf node_modules apps/*/node_modules packages/*/node_modules
git checkout asi-one-mobile-staging
pnpm i && pnpm build

# Go back to your branch
git checkout your-branch
cd apps/asi-one-native
pnpm dev
```

### Verify No Nested React

```bash
ls node_modules/react-native-css-interop/node_modules/react 2>/dev/null && echo "BAD: Nested React found" || echo "GOOD: No nested React"
```

### Clear Simulator State

If app is in a stuck state:

```bash
# Erase simulator content
xcrun simctl erase "iPhone 17 Pro"

# Or delete just the app
xcrun simctl uninstall booted ai.asione
```

### Auth0 Callback URL Issues

The iOS app uses scheme `ai.asione.auth0://`. Required callback URL format:

```
ai.asione.auth0://fetch-ai.us.auth0.com/ios/ai.asione/callback
```

---

## Key Rules

1. **NEVER run `pnpm i` on staging or feature branches** - Always install from `asi-one-mobile-staging`
2. **Keep node_modules when switching branches** - The correct React comes from mobile-staging
3. **Build native code after clean install** - Run `pnpm build` after `pnpm i`
4. **Always run `pnpm run orval` after switching back** - Regenerates API client types from the latest OpenAPI spec so hooks/types match the current backend
5. **Always run `pnpm build:chat` after orval** - Rebuilds the chat package against the current branch's code (staging or feature branch), not mobile-staging's
6. **Use `pnpm dev` not `npx expo start`** - Ensures correct environment

---

## Quick Reference

| Action | Command |
|--------|---------|
| Start dev server | `pnpm dev` |
| Regenerate API client | `pnpm run orval` |
| Build chat package | `pnpm build:chat` |
| Build iOS | `npx expo run:ios` |
| Build Android | `npx expo run:android` |
| Fix dependencies | `npx expo install --fix` |
| Check issues | `npx expo-doctor` |
| Clean node_modules | `rm -rf node_modules apps/*/node_modules packages/*/node_modules` |

---

## Integration

- Related to github-flow.md for branch management
- Auth0 config documented in slack thread C0999KNTC7M/p1769400565215749
