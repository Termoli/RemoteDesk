RemoteDeck v3.15
================

RemoteDeck is a local Windows host app with a mobile control surface for your
phone. It pairs by QR code and lets you control media, audio, quick-launch
apps, power actions, SignalRGB, FanControl profiles, iCUE helpers and selected
system status from the same private network.

RemoteDeck is designed as a local-first tool. It does not require a cloud
account, and the mobile UI can be saved as a PWA/Home Screen app.


Quick Start
-----------

1. Start `RemoteDeck\RemoteDeck.exe`.
2. Scan the QR code shown in the Windows host window.
3. Accept the local certificate warning if you use HTTPS, or use the local HTTP
   QR link for a faster first test.
4. Add the page to the phone Home Screen if you want a standalone PWA feel.
5. Once the phone is connected, the Windows host can minimize itself to the
   tray. Left click or double click the tray icon opens it again; right click
   shows Open, Show QR Code and Exit actions.


Windows Host
------------

- The built app lives in `RemoteDeck\RemoteDeck.exe`.
- The host is a native Win32 window, not a browser or Tkinter shell.
- It starts `server_v3.py` embedded and displays a large QR code for pairing.
- HTTP runs on port `8080`; HTTPS runs on port `8443`.
- Window size and position are saved and restored.
- Closing the window minimizes RemoteDeck to the tray instead of shutting down.
- The app is single-instance: starting it again opens the existing instance.
- The host status is intentionally sparse and cached to avoid excessive polling.
- Setup in the Windows app is the place for app paths, scans, security and
  server settings. The mobile app stays focused on remote controls.
- Native UI patches live in `patches/native_app.json`, so titles, accent
  colors, auto-tray and refresh intervals can be adjusted later without a full
  code rewrite.


Mobile Control Surface
----------------------

The mobile UI is served by the local Python server and can be installed as a
PWA on iOS and Android.

Main tabs:

- `Home`: connection, shutdown status, system values, volume, brightness and
  power actions.
- `Audio`: volume, mute, Windows output devices and global media keys.
- `Apps`: quick-launch apps, SignalRGB, iCUE, FanControl profiles and selected
  integration controls.
- `Setup`: mobile-safe status and shortcuts. PC paths and host-level settings
  remain in the Windows app.

The UI uses touch-down feedback, spring animations, haptic-style transitions,
toast feedback, value pop animations and SignalRGB-aware ambient backgrounds.
Reduced-motion and lower-power device modes are respected.


Power Controls
--------------

- Restart has been removed.
- Shutdown is scheduled only from a user-entered minute value.
- Setting a shutdown timer requires double confirmation.
- Timers are created as Windows Scheduled Tasks.
- Existing shutdown state is read back after server restarts.
- Cancel removes the scheduled task and also calls `shutdown /a`.


Audio, Media And Input
---------------------

- Volume and mute state are read back after each action.
- Windows output devices are listed and can be switched from the Audio tab.
- Media keys are sent as global Windows media keys first and fall back to
  APPCOMMAND only if needed.
- Supported global actions include previous, play/pause, next, stop, volume
  down, volume up and mute.
- YouTube helper keys send keyboard shortcuts to the active focused window.
- Mouse and keyboard control uses `/api/input`.
- Touchpad movement is coalesced client-side to avoid flooding the network.


Brightness
----------

- Internal displays are read and controlled through WMI.
- Multiple WMI displays are represented as devices; the displayed value is the
  average of active displays.


Integrations
------------

### SignalRGB

- OpenRGB support was removed.
- RemoteDeck first tries the local SignalRGB API at
  `http://127.0.0.1:16038/api/v1`.
- If the API is unavailable or restricted, RemoteDeck falls back to
  `SignalRgbLauncher.exe --url=... -silentlaunch-`.
- The UI exposes the mode as a state:
  - API mode: full control where available.
  - Launcher/URL mode: effect launching without brightness or canvas API.
  - Offline: SignalRGB actions are disabled.
- `api_requires_pro` is not treated as a fatal error. RemoteDeck marks the
  API-only areas but keeps launcher-based effects available.
- Installed effects are cached and merged with the bundled fallback library.
- Effect lists are refreshed sparingly to avoid overloading SignalRGB.

Default launcher path:

```text
%LOCALAPPDATA%\VortxEngine\SignalRgbLauncher.exe
```

### FanControl

- Default path: `C:\Program Files (x86)\FanControl\FanControl.exe`.
- RemoteDeck-owned profiles live in `fan_profiles/`:
  - `remote_silent.json`
  - `remote_balanced.json`
  - `remote_gaming.json`
- Profiles are applied through `FanControl.exe -c`.
- `active_profile` is read from FanControl's own cache file. If FanControl is
  not running, RemoteDeck falls back to the last profile name it sent.
- A UAC prompt is not treated as a failed action; the UI marks the action as
  sent and tells the user to confirm it on the PC.
- Temperature values prefer active FanControl curve sources where possible.
- NVIDIA GPU values can come from `nvidia-smi`; CPU and other sensor data can
  be matched through LibreHardwareMonitor/OpenHardwareMonitor WMI providers.

### iCUE And Lights

- `All Lights Off` calls SignalRGB lights-off first and then an optional
  configured iCUE lights-off command.
- The iCUE lights-off button is hidden when no external command is configured.
- iCUE does not expose a real "lights on again" API here; RemoteDeck offers an
  iCUE restart action for that state.
- iCUE Start/Stop stays separate so lighting can be switched off without
  forcing iCUE to close.


Performance And Caching
-----------------------

- Normal mobile status refreshes are throttled and paused when the page is in
  the background.
- Expensive system, audio, power and integration checks are cached server-side.
- Dependency checks are warmed in the background on host startup.
- FanControl temperatures use short TTLs.
- SignalRGB effect scans are cached and refreshed only when needed.
- Sliders debounce network writes.
- The native Windows app caches QR rendering, config and GDI resources to keep
  resize and paint operations smooth.


Security And Privacy
--------------------

- By default the server accepts loopback and private LAN addresses only.
- Pairing uses a QR token. Saved PWA/Home Screen shortcuts keep the token so
  the phone can reconnect after the server restarts.
- HTTPS certificates are generated per device in the writable data directory.
  Shared `cert.pem`/`key.pem` files are not shipped in release bundles.
- Runtime state, local certificates and logs should never be committed.


Configuration
-------------

Important keys in `remote_config.json`:

- `signalrgb_path`: path to `SignalRgbLauncher.exe`.
- `signalrgb_api_base`: local SignalRGB API base URL.
- `signalrgb_off_effect`: fallback off effect for launcher mode.
- `signalrgb_effects`: favorite effect names or IDs.
- `icue_path`: path to `iCUE.exe`.
- `icue_lights_off_command`: optional external command for iCUE lights off.
- `fancontrol_path`: path to `FanControl.exe`.
- `fan_profiles`: optional profile paths. Empty values fall back to bundled
  `fan_profiles/remote_*.json` files.
- `quick_launch`: list of app IDs, labels and commands.
- `ui.theme`: `dark`, `light` or `system`.
- `security.private_network_only`: allow only local/private networks.
- `security.allow_loopback`: allow access from the PC itself.


Installer And Release Preparation
---------------------------------

RemoteDeck is prepared for PyInstaller and Inno Setup:

- `BUILD_REMOTEDECK_EXE.bat` builds the PyInstaller `onedir` bundle.
- `installer/smoke_test_bundle.ps1` verifies required runtime files.
- `installer/RemoteDeck.iss` builds the Inno Setup installer.
- `installer/package_portable_zip.ps1` creates a portable ZIP without Inno.
- `installer/build_release.ps1` orchestrates bundle, portable ZIP, signing,
  installer build and release manifests.
- `.github/workflows/signpath-release.yml` prepares SignPath-signed GitHub
  releases: sign EXE, rebuild installer with the signed EXE, sign setup, write
  checksums and optionally create a draft GitHub Release.
- `docs/GITHUB_RELEASE.md` contains the publishing checklist.

End users should not need Python, pip, OpenSSL, Visual C++ runtime, Inno Setup
or PyInstaller. Those tools are needed only on the build machine.


Updates
-------

The app prepares `%LOCALAPPDATA%\RemoteDeck\updates` at runtime:

- `staged/` receives a future downloaded or copied bundle.
- `RemoteDeckApplyUpdate.cmd` waits for the running app to exit, copies staged
  files over the install folder and starts RemoteDeck again.
- Pairing state, local certificates, logs and runtime files are preserved.
- The update helper is launched hidden and detached.


Patchable UI
------------

- `ui/index.html` is preferred when present and valid.
- If the file is missing or invalid, the server falls back to the embedded UI
  inside `server_v3.py`.
- `patches/native_app.json` is used for native host UI tweaks.
- The goal is to keep design and animation changes patchable without forcing a
  full EXE rebuild.


PWA / Home Screen
-----------------

- `/manifest.webmanifest`, `/icon.svg`, `/apple-touch-icon.png` and
  `/icon-512.png` are served by the local server.
- iOS can add RemoteDeck to the Home Screen with the RemoteDeck icon.
- Android/Chrome receives a standalone PWA manifest with a maskable 512 px
  icon, theme color and stable app ID.


Notes
-----

Temperature data appears only when a supported provider exposes sensor data,
for example FanControl, NVIDIA tools, LibreHardwareMonitor or
OpenHardwareMonitor WMI providers.
