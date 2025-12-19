---
name: pnpm-upgrade
description: Guide for upgrading pnpm versions (e.g., v9 to v10+). Use when upgrading pnpm, handling pnpm breaking changes, or configuring pnpm after a major version upgrade.
---

# pnpm Upgrade Guide

## Upgrade Command

```bash
# Upgrade to latest version
pnpm self-update

# Upgrade to specific major version
pnpm self-update 10

# Upgrade to exact version
pnpm self-update 10.12.1
```

See: https://pnpm.io/cli/self-update

## Alternative Installation Methods

See: https://pnpm.io/installation

---

## Recommended New Settings (10.x)

### `allowBuilds` (10.26+)

Unified setting for controlling which packages can run lifecycle scripts. Replaces deprecated `onlyBuiltDependencies` and `ignoredBuiltDependencies`.

See: https://pnpm.io/npmrc#allowbuilds

### `strictDepBuilds` (10.3+)

Fail installation if dependencies have unreviewed build scripts.

See: https://pnpm.io/npmrc#strictdepbuilds

### `verify-deps-before-run` (10.0+)

Validate node_modules state before running scripts.

See: https://pnpm.io/npmrc#verifydepsbeforerun

### `blockExoticSubdeps` (10.26+)

Restrict transitive dependencies to trusted sources (registries, local paths, workspace links).

See: https://pnpm.io/npmrc#blockexoticsubdeps

### `trustPolicy` (10.21+)

Prevent installation of packages with downgraded trust levels.

See: https://pnpm.io/npmrc#trustpolicy

### Build Script Management Commands (10.1+)

```bash
pnpm ignored-builds    # See packages with blocked scripts
pnpm approve-builds    # Interactively approve packages
```

---

## Investigating Build Scripts Before Allowing

When pnpm blocks a package's lifecycle scripts, investigate before adding to `allowBuilds`:

### 1. Check what scripts the package defines

```bash
# View package.json scripts section
cat node_modules/<package-name>/package.json | grep -A 20 '"scripts"'

# Or fetch from npm registry
npm view <package-name> scripts
```

### 2. Read the actual script code

Common locations:
- `node_modules/<package-name>/scripts/`
- `node_modules/<package-name>/install.js`
- `node_modules/<package-name>/postinstall.js`
- Inline in package.json scripts field

### 3. Understand common script purposes

| Package Type | What Scripts Do | Risk if Blocked |
|--------------|-----------------|-----------------|
| Native modules (sharp, bcrypt, fsevents) | Compile C/C++ bindings via node-gyp | Package won't work at all |
| Bundled binaries (esbuild, swc, lightningcss) | Download platform-specific binary | Package won't work at all |
| Prisma, drizzle | Generate client code | ORM queries fail |
| Husky | Set up git hooks | Git hooks won't install |
| Electron | Download Electron binary | App won't run |
| node-sass | Compile libsass bindings | Sass compilation fails |
| sqlite3, better-sqlite3 | Compile SQLite bindings | Database won't work |
| canvas | Compile Cairo bindings | Canvas rendering fails |
| puppeteer | Download Chromium | Browser automation fails |

### 4. Check if prebuilt binaries exist

Some packages (esbuild, swc, lightningcss) ship prebuilt binaries as optional dependencies. If the postinstall just downloads binaries, blocking may cause fallback to slower JS implementations or complete failure.

### 5. Verify package authenticity

Before allowing scripts, verify the package:
```bash
# Check package info
npm view <package-name>

# Check recent publish history for suspicious activity
npm view <package-name> time
```

### 6. Decision framework

**Allow if:**
- Package is widely used and well-maintained
- Script compiles native code or downloads official binaries
- Package won't function without the script

**Block if:**
- Script does something unnecessary (telemetry, analytics)
- Package works fine without it
- Unknown/untrusted package

---

## Key Documentation

| Topic | URL |
|-------|-----|
| Release Announcements | https://github.com/orgs/pnpm/discussions/categories/announcements |
| All Releases | https://github.com/pnpm/pnpm/releases |
| Configuration (.npmrc) | https://pnpm.io/npmrc |
| Installation | https://pnpm.io/installation |
| Self Update | https://pnpm.io/cli/self-update |

---

## pnpm 10.0 Breaking Changes

Release notes: https://github.com/orgs/pnpm/discussions/8945

### Lifecycle Scripts Disabled by Default

Dependency install/postinstall scripts no longer run automatically. Native modules (esbuild, sharp, fsevents, bcrypt, node-sass, @swc/core, lightningcss, etc.) will fail to build.

Fix: https://pnpm.io/npmrc#onlybuiltdependencies (deprecated in 10.26+)

Fix (10.26+): https://pnpm.io/npmrc#allowbuilds

### Public Hoisting Disabled

Packages like eslint and prettier no longer auto-hoist to root node_modules. IDE plugins and shared configs may not resolve.

Fix: https://pnpm.io/npmrc#public-hoist-pattern

### `pnpm add --global pnpm` Blocked

This command now fails. Use `pnpm self-update` instead.

See: https://pnpm.io/cli/self-update

### `pnpm link` Behavior Changed

Now adds overrides to root package.json. Global linking no longer requires `-g` flag. In workspaces, affects all projects.

See: https://pnpm.io/cli/link

### Hash Algorithm Changed to SHA256

Lockfile hashes use SHA256. Peer dependency hashes, side effects cache keys, and pnpmfile checksums all changed. Regenerate lockfile after upgrade.

### Store Version Bumped to v10

Global store format changed. First install after upgrade will be slower as packages re-download.

### Fewer Environment Variables in Scripts

Only `npm_package_name`, `npm_package_version`, `npm_package_bin`, `npm_package_engines`, and `npm_package_config` are available during script execution.

### NODE_ENV No Longer Affects Install

All dependencies install regardless of `NODE_ENV=production`. Use `--prod` flag explicitly.

See: https://pnpm.io/cli/install

### `#` Character Escaped in node_modules

Directory names in `node_modules/.pnpm` now escape `#` characters. May affect tooling parsing these paths.

### `pnpm deploy` Restrictions

Only works in workspaces with `inject-workspace-packages=true`.

See: https://pnpm.io/cli/deploy

### `pnpm test` Parameter Passing

Parameters after `test` pass directly to scripts without requiring `--` prefix.

---

## pnpm 10.26 Semi-Breaking Changes

Release: https://github.com/pnpm/pnpm/releases/tag/v10.26.0

### Git Dependencies Require Explicit Approval

Git-hosted dependencies are now blocked from running `prepare` scripts unless explicitly allowed in `allowBuilds`.

### HTTP Tarball Integrity Hashes

Integrity hashes are now computed and stored in the lockfile for HTTP tarball dependencies.

---

## Deprecations

| Deprecated Setting | Replacement | Version |
|-------------------|-------------|---------|
| `onlyBuiltDependencies` | `allowBuilds` | 10.26+ |
| `ignoredBuiltDependencies` | `allowBuilds` | 10.26+ |

---

## Version Check

```bash
pnpm --version
```
