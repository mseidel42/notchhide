# NotchHide

A small macOS menubar app that hides the MacBook notch by adding a black bar across the top of your desktop wallpaper.

It reads the current wallpaper, renders a copy at the display's native resolution with a black bar the height of the menu bar, and sets that copy as the active wallpaper. The notch disappears into the bar.

Built as a replacement for TopNotch, which no longer works on newer macOS releases.

## Features

- Lives in the menubar. No Dock icon.
- Detects the current wallpaper and applies the black bar automatically.
- Supports both static images and dynamic HEIC wallpapers (solar, 24-hour time, and light/dark wallpapers).
- Optional auto-apply: re-processes when the wallpaper, Space, or display configuration changes.
- Works per display. Apply to the internal display only (default) or to all connected displays.
- Optional rounded corners: 28px black corners on all four corners of the wallpaper, with the top corners curving just below the black bar.
- Restores your original wallpaper at any time.
- Launch at login toggle.

## Requirements

- macOS 13 or later
- Xcode command line tools (Swift toolchain) to build from source

## Build and run

```sh
git clone https://github.com/drtaru/notchhide.git
cd notchhide
./build.sh
open build/NotchHide.app
```

`build.sh` runs `swift build -c release`, assembles `build/NotchHide.app` (including the `.icns` generated from `icons/appicon.png`), and ad-hoc signs it.

To use Launch at login, move `NotchHide.app` to `/Applications` first and run it from there. The login item points at the app's location.

Because the app is ad-hoc signed, Gatekeeper may block the first launch. Right-click the app and choose Open.

## Menu

| Item | What it does |
| --- | --- |
| Status line | Shows the last result or error |
| Hide Notch Now | Processes the current wallpaper immediately |
| Auto-apply on wallpaper change | Watches for changes and applies automatically. When off, nothing runs until you click Hide Notch Now |
| Apply to all displays | Checked: every connected display. Unchecked: internal display only. Unchecking restores external displays to their original wallpaper |
| Rounded corners | Adds 26px black rounded corners to all four corners. Toggling re-renders the current wallpaper from the saved original |
| Launch at login | Registers the app as a login item |
| Restore Original Wallpaper | Puts back the wallpaper that was active before NotchHide processed it |
| Quit | Quits the app |

## How it works

1. `NSWorkspace.desktopImageURL(for:)` returns the current wallpaper for each target screen.
2. The image is aspect-fill cropped to the screen's pixel size, and a black bar is drawn across the top. The bar height is the menu bar height (`frame.maxY - visibleFrame.maxY`).
3. For multi-frame dynamic wallpapers (HEIC), every frame is processed and encoded into a dynamic HEIC while preserving Apple's dynamic metadata (solar, 24h, and light/dark appearance schedule). Still images are encoded as PNG.
4. The result is written to `~/Library/Application Support/NotchHide/processed/` under a hashed filename. A new filename per source image forces macOS to refresh instead of using its cached copy.
5. `NSWorkspace.setDesktopImageURL(_:for:options:)` applies it. The original wallpaper path is stored so it can be restored.
6. Wallpapers that are already NotchHide output are skipped, so it never re-processes its own result.

macOS has no public notification for wallpaper changes, so auto-apply also polls every 3 seconds in addition to listening for Space and display changes. The check is a cheap URL comparison.

## Limitations

- Aerial and video wallpapers (macOS Sonoma/Sequoia video screensavers) are not supported because they are streamed video rather than still/multi-frame images. Dynamic HEIC wallpapers are fully supported.
- Wallpapers are set per Space. Each Space with its own wallpaper is processed when you switch to it (auto-apply) or click Hide Notch Now.
- The image is processed with aspect-fill. If your wallpaper was set to Fit or Center in System Settings, the layout changes slightly.
- External displays have no notch, so the bar there only matches the look of the internal display.

## Project layout

```
Package.swift                          Swift package manifest
build.sh                               Builds and bundles NotchHide.app
icons/                                 Menubar icons and app icon source
Sources/NotchHide/main.swift           App entry point
Sources/NotchHide/AppDelegate.swift    Menu, settings, watching, apply/restore
Sources/NotchHide/WallpaperProcessor.swift   Image rendering and caching
```

## Data stored

- Processed wallpapers in `~/Library/Application Support/NotchHide/processed/`. Only the 20 most recently used images are kept; older ones are deleted after each apply. Wallpapers currently set on a display are never deleted. A Space you haven't visited recently may lose its processed image and be re-processed when you switch to it.
- Preferences and the original wallpaper paths in the app's `UserDefaults`.

The app makes no network requests.

## License

[PolyForm Noncommercial 1.0.0](LICENSE). You may use, modify, and share this software for any noncommercial purpose. Selling it or using it commercially is not permitted. This is source-available, not an OSI-approved open source license.
