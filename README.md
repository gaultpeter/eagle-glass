<div align="center">

# 🦅 Eagle Glass

**A fast, native desktop IPTV player for Xtream Codes panels — Rust + `egui` + embedded libVLC.**

[![Release](https://img.shields.io/github/v/release/gaultpeter/eagle-glass?style=flat-square&color=8998ff&label=release)](https://github.com/gaultpeter/eagle-glass/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/gaultpeter/eagle-glass/total?style=flat-square&color=56d69a&label=downloads)](https://github.com/gaultpeter/eagle-glass/releases)
[![Stars](https://img.shields.io/github/stars/gaultpeter/eagle-glass?style=flat-square&color=8998ff&label=stars)](https://github.com/gaultpeter/eagle-glass/stargazers)
[![Forks](https://img.shields.io/github/forks/gaultpeter/eagle-glass?style=flat-square&label=forks)](https://github.com/gaultpeter/eagle-glass/network/members)
[![Issues](https://img.shields.io/github/issues/gaultpeter/eagle-glass?style=flat-square&label=issues)](https://github.com/gaultpeter/eagle-glass/issues)
[![Last commit](https://img.shields.io/github/last-commit/gaultpeter/eagle-glass?style=flat-square&label=last%20commit)](https://github.com/gaultpeter/eagle-glass/commits/main)
[![Top language](https://img.shields.io/github/languages/top/gaultpeter/eagle-glass?style=flat-square)](https://github.com/gaultpeter/eagle-glass)
[![License](https://img.shields.io/github/license/gaultpeter/eagle-glass?style=flat-square&color=56d69a)](LICENSE)

[![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D6?style=flat-square&logo=windows)](https://github.com/gaultpeter/eagle-glass/releases)
[![Rust](https://img.shields.io/badge/rust-stable-000000?style=flat-square&logo=rust)](https://www.rust-lang.org/)
[![UI](https://img.shields.io/badge/UI-egui-8998ff?style=flat-square)](https://github.com/emilk/egui)
[![Video](https://img.shields.io/badge/video-libVLC-ff8800?style=flat-square&logo=vlcmediaplayer&logoColor=white)](https://www.videolan.org/vlc/libvlc.html)

[**Download**](https://github.com/gaultpeter/eagle-glass/releases/latest) · [**Build from source**](#build-from-source) · [**CLI**](#cli) · [**Report a bug**](https://github.com/gaultpeter/eagle-glass/issues)

</div>

---

Eagle Glass is a **power-user-first** live-TV player: a small, snappy native window that logs into your
**Xtream Codes** panel, browses the whole catalogue, and plays channels with an **embedded libVLC**
surface (video renders *inside* the app - no separate VLC window popping up).

## Features

- **Browsable tree** - `Section -> category -> channel` in the sidebar, with per-category channel counts.
- **Embedded player** - video renders inside the app via a libVLC child surface; switching channels replaces the stream in place.
- **True 16:9** player pane (letterboxed when needed).
- **Sleek controls** - play/pause, volume, fullscreen - with a red **LIVE** badge.
- **Video-only fullscreen** - `F11`, the button, or **double-click** the video (double-click again to exit).
- **Keyboard driven** - `Space` play/pause, `M` mute, `F` fullscreen, `Up`/`Down` volume.
- **First-run wizard** + connection **pill** and a settings dialog; credentials are remembered.
- **CLI mode** for scripting (`categories`, `list`, `url`, `m3u`, `epg`).
- Clean dark theme, no clutter.

## Download

Grab the latest `eglass.exe` from the [**Releases**](https://github.com/gaultpeter/eagle-glass/releases/latest) page.

> [!IMPORTANT]
> **64-bit VLC must be installed** (Eagle Glass loads `libvlc.dll` at runtime - it does not bundle it).
> Install [VLC](https://www.videolan.org/vlc/) or point `EG_VLC_DIR` at a folder containing `libvlc.dll`.

## Quick start

1. Run `eglass.exe`.
2. On first launch a **welcome wizard** asks for your panel **Server**, **Username** and **Password**.
   They're saved to `%APPDATA%\EagleGlass\config.txt` and reused automatically next time.
3. Pick a category in the sidebar, expand it, and click a channel to play.
4. Double-click the video (or press `F11`) for fullscreen.

> [!WARNING]
> The saved config file is **plaintext**. Treat `%APPDATA%\EagleGlass\config.txt` like a secret, and don't
> commit or share it.

## Build from source

**Requirements:** Windows 10/11, [Rust](https://rustup.rs/) (stable), 64-bit VLC installed.
(Visual Studio C++ Build Tools are needed for the MSVC toolchain.)

```bash
git clone https://github.com/gaultpeter/eagle-glass.git
cd eagle-glass
cargo build --release
```

Output: `target\release\eglass.exe`

## Keyboard shortcuts

| Key | Action |
|-----|--------|
| `Space` | Play / pause |
| `M` | Mute / unmute |
| `F` | Toggle fullscreen |
| `Esc` | Exit fullscreen |
| `Up` / `Down` | Volume up / down |
| double-click video | Toggle fullscreen |

## Environment variables

| Variable | Purpose |
|----------|---------|
| `EG_SERVER` / `EG_USER` / `EG_PASS` | Override the saved connection (auto-connects when user + pass are set). |
| `EG_AUTOPLAY` | Stream id to auto-play once the Live list loads. |
| `EG_VLC_DIR` | Folder containing `libvlc.dll` (defaults to the installed VLC). |

## CLI

```
eglass.exe categories
eglass.exe list live [category_id]
eglass.exe url  live <stream_id>
eglass.exe play live <stream_id>     # CLI play uses an external VLC process
eglass.exe m3u playlist.m3u
eglass.exe epg
```

## How the video works

`src/video.rs` loads `libvlc.dll` at runtime (via [`libloading`](https://crates.io/crates/libloading)),
creates a child Win32 window inside the app window, and points libVLC at it with
`libvlc_media_player_set_hwnd`. The child window is repositioned every frame to match the `egui`
video rectangle. Because that surface is a native child window, `egui` can't paint over it - so the
control bar sits directly beneath the video, and the settings dialog temporarily hides the video while open.

## Project layout

| File | Purpose |
|------|---------|
| `src/xtream.rs` | Xtream client: login, categories, items, episodes, URL builders, M3U/EPG |
| `src/video.rs` | libVLC FFI + embedded Win32 child window |
| `src/main.rs` | `egui` GUI + CLI |

## Notes & limitations

- **Windows only** - the embedded video relies on a Win32 child window.
- Requires a **64-bit VLC** install for `libvlc.dll`.
- Blocking HTTP (`ureq`) runs on worker threads; the UI stays responsive.
- `max_connections` is enforced by your panel; on many panels concurrent streams from one IP count as one.
- Live streams have no seek bar (no timeshift).

## License

[MIT](LICENSE) © 2026 Peter Gault
