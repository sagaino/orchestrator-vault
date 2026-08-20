---
title: "Multi-Flavor Build Parameterization and Runtime Environment Bootstrapping Pipeline"
type: pattern
tags: [pattern, frontend, devops, electron, build-pipeline, environment-bootstrapping, electron-builder]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# Multi-Flavor Build Parameterization and Runtime Environment Bootstrapping Pipeline

Multi-Flavor Build Parameterization and Runtime Environment Bootstrapping Pipeline with isolated product identifiers and cascading dotenv resolution.

## 1. Overview & Architecture

A multi-flavor DevOps packaging and runtime bootstrapping architecture that parameterizes build targets (development vs production across Windows/Mac/Linux) through a custom build runner script, generates isolated application identifiers, and dynamically resolves cascading environment files at runtime across packaged desktop resource directories.

## 2. Implementation & Code Structure

scripts/
  └── build-electron.mjs             # CLI orchestrator emitting build-info.json and executing multi-target builds
electron-builder.config.cjs          # Dynamic builder config resolving app ID, release dir, and resource bundling
src/main/
  └── load-env.ts                    # Cascading runtime environment loader traversing candidate directories

## 3. Key Implementation Points

- Compile-time manifest synthesis via build-electron.mjs injecting appEnv into build-info.json.
- Conditional electron-builder target matrix generating parameterized artifact names, release folders, and Windows NSIS uninstaller display names.
- Multi-tiered runtime bootstrapping in load-env.ts checking process.resourcesPath before process.cwd and dotenv cascading.

## 4. Code Examples

### Build runner script that parameterizes the environment mode, emits compile manifest, and spawns electron-vite and electron-builder.

```typescript
// scripts/build-electron.mjs
const targetArg = (process.argv[2] ?? '').toLowerCase()
const modeArg = (process.argv[3] ?? 'production').toLowerCase()

const targetFlags = { win: '--win', mac: '--mac', linux: '--linux' }
const targetFlag = targetFlags[targetArg]

const env = {
  ...process.env,
  APP_ENV: modeArg,
  NODE_ENV: modeArg,
}

// Persist compile-time flavor manifest
writeFileSync(
  resolve(rootDir, 'build-info.json'),
  JSON.stringify({ appEnv: modeArg }, null, 2),
)

// Orchestrate electron-vite bundling followed by electron-builder packaging
await runCommand(process.execPath, [electronViteCli, 'build'], env)
await runCommand(
  process.execPath,
  [electronBuilderCli, targetFlag, '--config', 'electron-builder.config.cjs'],
  env,
)
```

### Dynamic electron-builder configuration adapting app identifiers, release directories, and bundling extra binary resources.

```javascript
// electron-builder.config.cjs
const isDev = process.env.APP_ENV === 'development'

const productName = isDev ? 'Identify Verification Kit Dev' : 'Identify Verification Kit'
const appId = isDev ? 'com.identitykit.dashboard.dev' : 'com.identitykit.dashboard'
const packageName = isDev ? 'identify-verification-kit-dev' : 'identify-verification-kit'
const artifactPrefix = isDev ? 'identify-verification-kit-dev' : 'identify-verification-kit'

module.exports = {
  appId,
  productName,
  extraMetadata: { name: packageName },
  artifactName: `${artifactPrefix}-\${version}-\${arch}.\${ext}`,
  directories: {
    output: isDev ? 'release/dev' : 'release/prod',
  },
  extraResources: [
    { from: 'src/renderer/public/sam.wasm', to: 'sam.wasm' },
    { from: 'build-info.json', to: 'build-info.json' },
  ],
  win: { icon: 'public/logo-ina.ico' },
  nsis: {
    include: 'build/installer.nsh',
    shortcutName: productName,
    uninstallDisplayName: `${productName} \${version}`,
  },
}
```

### Hierarchical runtime environment bootstrapping resolving build mode from packaged assets, cwd, and environment variables.

```typescript
// src/main/load-env.ts
const readBuildMode = (): string | undefined => {
  const candidatePaths = [
    resolve(process.resourcesPath, 'build-info.json'),
    resolve(process.cwd(), 'build-info.json'),
  ]

  for (const candidatePath of candidatePaths) {
    if (!existsSync(candidatePath)) continue
    try {
      const parsed = JSON.parse(readFileSync(candidatePath, 'utf-8')) as { appEnv?: string }
      if (parsed.appEnv) return parsed.appEnv
    } catch {
      continue
    }
  }
  return undefined
}

export const resolvedAppEnv =
  readBuildMode() ??
  process.env.APP_ENV ??
  process.env.NODE_ENV ??
  (process.env.ELECTRON_RENDERER_URL ? 'development' : 'production')

const isProductionMode = resolvedAppEnv === 'production'
const envFileNames = isProductionMode ? ['.env.production', '.env'] : ['.env']

const envBaseDirs = [
  process.cwd(),
  resolve(__dirname, '../../'),
  resolve(__dirname, '../../../'),
  process.resourcesPath,
]

const envCandidatePaths = envBaseDirs.flatMap((baseDir) =>
  envFileNames.map((fileName) => resolve(baseDir, fileName))
)

const loadedPaths = new Set<string>()
for (const envPath of envCandidatePaths) {
  if (loadedPaths.has(envPath)) continue
  if (!existsSync(envPath)) continue
  dotenvConfig({ path: envPath, override: false })
  loadedPaths.add(envPath)
}
```

## 5. Considerations & Best Practices

- build-info.json must be declared as an extraResource in electron-builder so packaged production apps can inspect their build flavor at runtime via process.resourcesPath.
- Candidate paths for .env resolution must prioritize process.cwd() and process.resourcesPath without overwriting already loaded environment keys (override: false).
- Different app IDs (com.identitykit.dashboard.dev vs prod) enable side-by-side installations of development and production builds on client workstations.

## 6. Related Knowledge

- Electron Vite Configuration Pipeline
- Multi Flavor Desktop Packaging

## 7. Source

- Harvest 1787127443075 F8fd0716.json
