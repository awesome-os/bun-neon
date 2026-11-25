# `bun-neon` — Zero-JSC Bun Compatibility Layer for Awesome-OS

`bun-neon` provides a lightweight, erasable-TypeScript Bun-compatible runtime surface (`globalThis.Bun`, `bun:sqlite`, `bun:test`, `bun:ffi`, and the `bun-neon` CLI) powered natively by Awesome-OS, Node.js built-ins (`node:sqlite`, `node:test`, `node:crypto`, `node:http`), and [`sectium-node-hook.mts`](../b8g-runtime/sectium-node-hook.mts) — with **zero JavaScriptCore (JSC)** binary overhead and **zero external npm dependencies**.

---

## Usage

Invoke `bun-neon` through the workspace `bin/` atomic triplet wrappers:

```powershell
# PowerShell
.\bin\bun-neon.ps1 --version
.\bin\bun-neon.ps1 -e "console.log(Bun.version)"
.\bin\bun-neon.ps1 run .\script.mts
.\bin\bun-neon.ps1 test sections/bun-neon/bun-neon.test.mts
.\bin\bun-neon.ps1 install
.\bin\bun-neon.ps1 add <pkg>
.\bin\bun-neon.ps1 x <pkg> [args...]
```

```cmd
:: cmd.exe
.\bin\bun-neon.cmd --version
.\bin\bun-neon.cmd -e "console.log(Bun.version)"
```

---

## CLI Commands

| Command | Behavior |
| :--- | :--- |
| `--version`, `-v` | Prints `1.2.0-neon (bun-neon on Awesome-OS; zero JSC)` and exits `0`. |
| `--help`, `-h` | Prints CLI usage information and exits `0`. |
| `-e <code>`, `--eval <code>` | Evaluates inline JS/TS code with `globalThis.Bun` and Sectium hooks active. |
| `run <file> [...args]` / `<file>` | Runs a `.ts`/`.mts`/`.js` script directly with `globalThis.Bun` + `sectium-node-hook.mts`. |
| `test [...files]` | Spawns `node --import <sectium-node-hook> --test ...files` (`shell: false`). |
| `install [...args]` / `add [...args]` | Routes cleanly to `pnpm` via `@awesome-os/bin-shim` (never `npm`/`npx`). |
| `x <pkg> [...args]` | Routes cleanly to `pnx` via `@awesome-os/bin-shim` (never `npx`). |

---

## API Compatibility Matrix

| Module / Surface | Key Exports | Implementation Backing |
| :--- | :--- | :--- |
| [`index.mts`](index.mts) | Barrel re-export of all `bun-neon` surfaces | Erasable TypeScript (zero deps) |
| [`bun-global.mts`](bun-global.mts) | `Bun` (`file`, `write`, `serve`, `spawn`, `spawnSync`, `hash`, `CryptoHasher`, `password`, `deepEquals`, `escapeHTML`, `nanoseconds`, `sleep`, `which`, `env`, `version`), `installBunNeonGlobal` | `node:fs`, `node:http`, `node:child_process`, `node:crypto`, `node:util` |
| [`bun-sqlite.mts`](bun-sqlite.mts) | `Database` (`query`, `prepare`, `exec`, `run`, `transaction`, `close`) | Native `node:sqlite` (`DatabaseSync`) |
| [`bun-test.mts`](bun-test.mts) | `describe`, `test`, `it`, `beforeAll`, `afterAll`, `beforeEach`, `afterEach`, `expect`, `mock`, `spyOn` | Native `node:test` + `node:assert/strict` |
| [`bun-ffi.mts`](bun-ffi.mts) | `FFIType`, `suffix`, `ptr`, `CString`, `toArrayBuffer`, `dlopen`, `ghostFFI`, `getGhostCasDir` | Native buffer/pointer utilities, FFI ABI & GhostModuleRegistry CAS hydration |
| [`cli.mts`](cli.mts) | `bun-neon` CLI entrypoint (`run`, `test`, `-e`, `install`, `add`, `x`) | `node:vm`, `node:child_process` (`shell: false`), `sectium-node-hook.mts` |

---

## Ghost-Driven Versioned FFI (`ghostFFI` + `GhostModuleRegistry`)

Resolve versioned, SHA-256-verified native shared libraries directly from the machine CAS registry (`awesome-os/b8g-ghost`) with zero `node-gyp` or manual installs:

```ts
import { ghostFFI } from './bun-ffi.mts';

const display = await ghostFFI('awesome-os/sectium-browser@v0.5.0-baseline', {
  lib: 'libsectium_display',
  sha256: '50badeec9a1882ea6997b327f207bc91366561c5e3bff56572cfb1e7fc85b923',
  symbols: {
    sectium_display_init: { args: ['ptr'], returns: 'i32' },
  },
});
// Cached in %LOCALAPPDATA%/AwesomeOS/cas/<sha256>/ (or ~/.local/share/awesome-os/cas/<sha256>/)
display.dispose();
```

---

## Verification

```powershell
.\bin\node.ps1 --test sections/bun-neon/bun-neon.test.mts
.\bin\b8g-web-doc.ps1 sections/bun-neon/README.md
```

## Quick Install & Run (Windows / Linux / macOS)

```sh
# Install / hydrate from v11 CAS via holo-pm
holo-pm install repository://github.com/awesome-os/bun-neon@main

# Or project & run directly via holo-project (0-byte hardlink)
holo-project "project://cpp/bun-neon?from=repository://github.com/awesome-os/bun-neon@main"
```
