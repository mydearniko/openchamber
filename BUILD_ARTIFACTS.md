# Local Artifact Builds

This note records the local process used to build distributable artifacts from this fork.

## Prerequisites

Install workspace dependencies from the lockfile:

```bash
bun install --frozen-lockfile
```

When cross-building the Windows installer from Linux, Electron Builder needs Wine. On Ubuntu, install both the `wine` wrapper and 32-bit support because Electron Builder runs 32-bit Windows tooling during NSIS packaging:

```bash
dpkg --add-architecture i386
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y wine wine64 wine32:i386
apt-get clean
```

## VS Code Extension

Build and package the VSIX:

```bash
bun run --cwd packages/vscode build
bun run --cwd packages/vscode package
```

Output:

```text
packages/vscode/openchamber-<version>.vsix
```

## Windows Installer From Linux

Build the Electron web assets, bundle the Electron main process, rebuild native modules for Electron, then run Electron Builder for Windows x64:

```bash
bun run --cwd packages/electron build:web-assets
bun run --cwd packages/electron bundle:main
bun run --cwd packages/electron rebuild:native
CSC_IDENTITY_AUTO_DISCOVERY=false node packages/electron/scripts/package.mjs --win --x64 --publish never -c.win.signAndEditExecutable=false
```

`-c.win.signAndEditExecutable=false` is used for this local unsigned Linux cross-build to avoid Wine/rcedit failures while still producing the NSIS installer.

Output:

```text
packages/electron/dist/OpenChamber-<version>-win-x64.exe
packages/electron/dist/OpenChamber-<version>-win-x64.exe.blockmap
packages/electron/dist/win-unpacked/OpenChamber.exe
```

If packaging fails with `System ERROR: E_FAIL` from `7za`, check disk space first; NSIS packaging needs enough free space to compress `packages/electron/dist/win-unpacked`.
