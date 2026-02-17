# Desktop Environment Roadmap (starting from existing D3D/D2D taskbar + shell integrations)

Assumptions:
- We already have: taskbar, SHAppBar, NotifyIcon/Tray backend, flyouts, modules, renderer, config/theme loader (JSON), system stats.
- Goal: evolve into a full DE with WebView2-based theming/customization *without* regressing stability.

Legend:
- [ ] todo
- [x] done
- (ETA) rough solo-dev time assuming ~15–25 hrs/week

---

## Phase 0 — Re-scope & architecture locks (ETA: 3–7 days)
- [ ] Define “Explorer relationship” strategy:
  - [ ] Default: run alongside Explorer (safe mode)
  - [ ] Optional: shell replacement (HKCU\...\Winlogon\Shell) + rollback tool
- [ ] Define process model:
  - [ ] Single host process + multiple windows (taskbar/start/desktop/flyouts)
  - [ ] Crash watcher / auto-relaunch (separate tiny watchdog exe or scheduled task)
- [ ] Define UI tech split:
  - [ ] Keep native D3D/D2D for performance-critical chrome (taskbar frame, hit-testing)
  - [ ] Embed WebView2 for “content surfaces” (start menu content, widgets, flyouts, theming UI)
- [ ] Decide theme packaging format v1:
  - [ ] theme.json (metadata, token defaults)
  - [ ] tokens.json (colors, spacing, radii, fonts)
  - [ ] layout.json (module placement)
  - [ ] web/ (html/css/js assets)
- [ ] Create a “Recovery Contract”:
  - [ ] Hotkey to restore Explorer UI (even if your shell is primary)
  - [ ] Auto-revert on N crashes at startup
  - [ ] Safe theme fallback if theme fails to load

---

## Phase 1 — WebView2 foundation integrated into current codebase (ETA: 1–2 weeks)
Target folders touched: App/, Renderer/, UI/, Config/
- [ ] Add WebView2 runtime bootstrap + version detection
  - [ ] Graceful message if runtime missing + install link
- [ ] Create WebViewHost abstraction
  - [ ] UI/WebViewHost.h/.cpp (init, navigation, postMessage, resource mapping)
- [ ] Create “UI Surface” contract (native or web)
  - [ ] UI/ISurface.h (Show/Hide/Resize/SetTheme/HandleInput)
- [ ] Wire WebViewHost into one existing flyout (VolumeFlyout) as a spike
  - [ ] Keep native flyout available behind a flag
- [ ] Add message bridge: native -> web and web -> native
  - [ ] Commands: open settings, change volume, network toggle, launch app, etc.
- [ ] Add telemetry-free perf counters (internal):
  - [ ] WebView process count, mem estimate, frame time, input latency

---

## Phase 2 — Desktop window + wallpaper + basic widgets (ETA: 2–4 weeks)
Target folders: App/, UI/, WindowManager/, WorkspaceManager/, Modules/
- [ ] Implement DesktopWindow (behind icons)
  - [ ] UI/DesktopWindow.h/.cpp (WS_EX_TOOLWINDOW?; set to bottom; multi-monitor)
- [ ] Wallpaper pipeline
  - [ ] Simple static image
  - [ ] Optional: video/gif later
- [ ] Widget host v1
  - [ ] Option A: Widgets are Modules rendered in WebView2
  - [ ] Option B: Widgets are native modules (existing Modules/) placed on desktop
- [ ] Multi-monitor desktop surfaces
  - [ ] One DesktopWindow per monitor
  - [ ] Correct DPI scaling + taskbar offsets
- [ ] Workspace/virtual desktop awareness
  - [ ] Hide/show widgets per workspace (optional)
- [ ] Settings flags:
  - [ ] EnableDesktop
  - [ ] EnableWidgets
  - [ ] EnableWebFlyouts (per component)

---

## Phase 3 — Start menu & launcher (ETA: 2–4 weeks)
Target folders: UI/, Services/, Config/
- [ ] Replace/augment UI/MainMenu.* with WebView2-based Start surface
  - [ ] Search box + results
  - [ ] Pinned apps + recent
- [ ] App indexing service
  - [ ] Services/AppIndex.h/.cpp (Start Menu shortcuts, installed apps, PATH)
- [ ] Fast search (no web, no Bing)
  - [ ] Prefix matching + fuzzy (optional)
- [ ] Invocation bridge
  - [ ] Web “launch” -> native ProcessStart + UWP activation support
- [ ] Keyboard-first UX
  - [ ] Win key opens
  - [ ] Arrow navigation
  - [ ] Enter launches
  - [ ] Esc closes

---

## Phase 4 — Theme system v2 (tokens + layouts + modules) (ETA: 2–3 weeks)
Target folders: Config/, Modules/, Renderer/, UI/
- [ ] Upgrade Config/ThemeLoader to support:
  - [ ] tokens.json (design tokens)
  - [ ] layout.json (surface composition)
  - [ ] capabilities.json (requires metrics, requires audio capture, etc.)
- [ ] Add token propagation:
  - [ ] Native renderer consumes tokens (fonts, colors, corner radius)
  - [ ] Web surfaces receive tokens via injected CSS variables
- [ ] Add theme validation:
  - [ ] JSON schema validation (basic)
  - [ ] Missing asset detection
  - [ ] Safe fallback theme on failure
- [ ] “Apply Theme” transaction
  - [ ] Stage -> validate -> apply -> rollback if crash within X seconds

---

## Phase 5 — Pack/bundler MVP (install/uninstall/revert) (ETA: 1–2 weeks)
Target folders: Config/, Services/
- [ ] Define Pack format (.zip or .depack)
  - [ ] manifest.json
  - [ ] themes/
  - [ ] plugins/ (future)
  - [ ] config overrides
- [ ] Implement pack install
  - [ ] Copy files into known dirs
  - [ ] Update config pointers
  - [ ] Keep backup snapshot for revert
- [ ] Implement pack uninstall/revert
  - [ ] Restore previous config snapshot
  - [ ] Remove pack assets (only those installed by pack)

---

## Phase 6 — Reliability hardening (ETA: 2–4 weeks)
Target folders: App/, WindowManager/, Services/, UI/
- [ ] Watchdog / recovery executable
  - [ ] Detect crash loops
  - [ ] Auto-launch Explorer + revert shell reg on repeated failure
- [ ] “Safe mode boot”
  - [ ] Hold Shift at startup -> disable web surfaces + load default theme
- [ ] Module isolation rules
  - [ ] Rate-limit expensive metrics updates
  - [ ] Move heavy services off UI thread
- [ ] Update strategy
  - [ ] In-app “check update” (no auto-update required)
  - [ ] Signed releases (optional but recommended)

---

## Phase 7 — Pro tooling (monetizable) (ETA: 4–8 weeks)
This is after you have a stable alpha that users can daily-drive.
- [ ] Visual layout editor (WebView2 app)
  - [ ] Drag/drop modules/widgets
  - [ ] Property inspector (bound to tokens/layout)
  - [ ] Undo/redo
- [ ] Live preview + apply/rollback
  - [ ] Preview in a sandbox surface first
  - [ ] Promote to active on confirmation
- [ ] Theme dev tools
  - [ ] Inspector overlay (CSS vars / layout boxes)
  - [ ] Performance warnings (overdraw, update rate too high)

---

## Phase 8 — Optional cloud backup/sync (ETA: 4–8 weeks, can defer indefinitely)
- [ ] Export/import local backup first (free)
  - [ ] Single archive of configs + themes + packs
- [ ] Cloud sync v1 (paid)
  - [ ] End-to-end encryption
  - [ ] Restore-on-new-PC flow
  - [ ] Opt-in only, clear privacy story

---

# Recommended sequencing (fastest path to “wow”)
- [ ] Spike WebView2 in ONE flyout (Volume) -> prove bridge + theming pipeline
- [ ] DesktopWindow + wallpaper -> visible “DE” legitimacy
- [ ] Web Start menu -> biggest perceived value
- [ ] Theme tokens v2 -> enables real theming without rewriting everything
- [ ] Bundler/packs -> community growth + distribution
- [ ] Reliability hardening -> trust
- [ ] Then monetization: visual editor + preview

---

# Exit criteria per milestone (definition of done)
- [ ] Milestone A: Web flyout works with token theming and no input bugs
- [ ] Milestone B: Desktop + wallpaper stable across multi-monitor/DPI
- [ ] Milestone C: Start menu launches apps reliably, keyboard-first
- [ ] Milestone D: Theme apply is transactional with rollback
- [ ] Milestone E: Pack install/uninstall is reversible and safe
- [ ] Milestone F: Crash loop recovery always restores usability
