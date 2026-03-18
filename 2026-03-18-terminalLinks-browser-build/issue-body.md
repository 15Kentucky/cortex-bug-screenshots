========================
Issue Draft #1
Title: [BUG] [v0.1.0] Production build bundles Node-only fs/path into terminal link detection, causing browser externalization warnings (tested commit: 781b6da)

Project:
- ide

Description:
- `npm run build` on `main` succeeds with browser-bundle warnings showing that `src/utils/terminalLinks.ts` imports Node-only `fs` and `path` APIs into the client build.
- The terminal UI imports this module through `TerminalsContext`, so browser-side terminal link detection depends on APIs that Vite replaces with `__vite-browser-external` shims.
- This matters because file-link detection in terminal output can degrade or break in production builds even though the build exits successfully.

Error Message:
- `[plugin vite:resolve] Module "fs" has been externalized for browser compatibility, imported by "src/utils/terminalLinks.ts".`
- `[plugin vite:resolve] Module "path" has been externalized for browser compatibility, imported by "src/utils/terminalLinks.ts".`
- `src/utils/terminalLinks.ts (205:12): "isAbsolute" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".`
- `src/utils/terminalLinks.ts (274:27): "promises" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".`

Debug Logs:
- Repro command:
- `npm.cmd run build`
- Relevant excerpt:
```text
[plugin vite:resolve] Module "fs" has been externalized for browser compatibility, imported by "src/utils/terminalLinks.ts".
[plugin vite:resolve] Module "path" has been externalized for browser compatibility, imported by "src/utils/terminalLinks.ts".
src/utils/terminalLinks.ts (205:12): "isAbsolute" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".
src/utils/terminalLinks.ts (207:24): "resolve" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".
src/utils/terminalLinks.ts (268:12): "isAbsolute" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".
src/utils/terminalLinks.ts (270:24): "resolve" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".
src/utils/terminalLinks.ts (274:27): "promises" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".
src/utils/terminalLinks.ts (283:27): "resolve" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".
src/utils/terminalLinks.ts (285:31): "promises" is not exported by "__vite-browser-external", imported by "src/utils/terminalLinks.ts".
```

System Information:
- OS: Microsoft Windows 11 Pro 10.0.26200
- CPU/RAM: Intel(R) Core(TM) i7-8665U CPU @ 1.90GHz (4 cores / 8 logical), 15.78 GB RAM
- GPU (if relevant): N/A
- Node version: v24.14.0
- npm version: 11.9.0
- Rust version: N/A in this environment
- App version: 0.1.0 (package.json), 0.1.0 (src-tauri/tauri.conf.json)
- Git commit short SHA: 781b6da

Screenshots:
- Evidence type: Screenshot(s)
- What to capture:
- Build-output screenshot showing the Vite browser-externalization warnings for `src/utils/terminalLinks.ts`.
- Code screenshot showing Node-only imports in `src/utils/terminalLinks.ts` and the browser-side import path from `src/context/TerminalsContext.tsx`.
- How to capture (include OS steps):
- Windows: `Win+Shift+S` for screenshot, `Win+Alt+R` for video.
- macOS: `Shift+Cmd+4` for screenshot, `Shift+Cmd+5` for video.
- Linux: `PrtSc` / `Shift+PrtSc` for screenshot, `Ctrl+Alt+Shift+R` or OBS for video.
- When to capture:
- At step 4 when the build warnings print.
- At step 5 when the code lines are visible.
- Attach instructions:
- Drag and drop `.png/.jpg` or `.mp4` into the GitHub issue “Screenshots” field.
- Captured evidence:
- Build warnings:
  ![build-warning](https://raw.githubusercontent.com/15Kentucky/cortex-bug-screenshots/main/2026-03-18-terminalLinks-browser-build/build-warning.png)
- Code evidence:
  ![code-evidence](https://raw.githubusercontent.com/15Kentucky/cortex-bug-screenshots/main/2026-03-18-terminalLinks-browser-build/code-evidence.png)

Steps to Reproduce:
1. `git clone https://github.com/CortexLM/cortex-ide.git`
2. `cd cortex-ide`
3. `npm.cmd ci`
4. Start recording now, then run: `npm.cmd run build`
5. Observe Vite warnings that `src/utils/terminalLinks.ts` imports browser-externalized `fs`/`path` APIs.
6. Take screenshot now of the build output.
7. Open `src/utils/terminalLinks.ts` and `src/context/TerminalsContext.tsx` at the referenced lines.
8. Take screenshot now of the code evidence.

Expected Behavior:
- The production client build should not bundle terminal-link code that depends on Node-only `fs` and `path` modules.
- Browser-side terminal link detection should rely on browser-safe or Tauri-safe APIs only.

Actual Behavior:
- The client build emits browser-externalization warnings for `fs` and `path` from `src/utils/terminalLinks.ts`.
- `terminalLinks` still gets pulled into the browser terminal UI via `TerminalsContext`, leaving file-link behavior dependent on missing browser exports.

Additional Context:
- Severity: High
- Frequency: Always on `npm run build` for the tested commit.
- Workarounds: None upstream; requires splitting browser-safe detection from file-system-backed resolution or moving path resolution behind Tauri APIs.
- Suspected cause + file(s)/component(s):
- `src/utils/terminalLinks.ts:6`
- `src/utils/terminalLinks.ts:7`
- `src/utils/terminalLinks.ts:205`
- `src/utils/terminalLinks.ts:274`
- `src/context/TerminalsContext.tsx:18`
- `src/context/TerminalsContext.tsx:2342`
- Duplicate check: no matching repo issue found for targeted searches (`terminalLinks __vite-browser-external fs path`, `terminal link detection browser build`, `workspace trust open-settings no-op`) at time of filing.
========================
