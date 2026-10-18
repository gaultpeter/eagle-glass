# Eagle Glass

A desktop IPTV player for **Xtream Codes** panels, written in Rust.

- **Xtream Codes client** (login, categories, live/VOD/series, URL builders, M3U/EPG).
- **Native GUI** with `eframe` / `egui`.
- **Embedded libVLC** video surface — the video renders *inside* the app window, not a separate VLC window.
- Browsable **section → category → channel** tree in the sidebar.
- 16:9 player with Twitch-style controls (play/pause, volume, fullscreen) and a **LIVE** badge.
- **First-run wizard** to enter your panel details; they are stored locally and reused on later launches. A connection pill + **⚙ settings** dialog let you edit them any time.
- CLI mode for scripting.

## Requirements
- **Windows** (the embedded video uses a Win32 child window).
- 64-bit **VLC** installed (provides `libvlc.dll`), or set `EG_VLC_DIR`.
- Rust toolchain (stable).

## Build
```
cargo build --release
```
Output: `target\release\eglass.exe`

## Run
```
eglass.exe
```

On first run you'll get a small **welcome wizard** — enter your panel **Server**, **Username** and **Password**. They are saved to `%APPDATA%\EagleGlass\config.txt` (**plaintext — treat that file as a secret**) and reused automatically afterwards. Edit them any time via the **⚙** button in the top bar.

### Keyboard
| Key | Action |
|-----|--------|
| `Space` | Play / pause |
| `M` | Mute / unmute |
| `F` | Fullscreen |
| `Esc` | Exit fullscreen |
| `↑` / `↓` | Volume |

Fullscreen is video-only; exit with `F`, `Esc`, or double-click the video.

### Environment variables
| Var | Purpose |
|-----|---------|
| `EG_SERVER` / `EG_USER` / `EG_PASS` | override the saved connection (auto-connects if user + pass are set) |
| `EG_AUTOPLAY` | stream id to auto-play once the Live list loads |
| `EG_VLC_DIR` | folder containing `libvlc.dll` (default: the installed VLC) |

## Video engine
`src/video.rs` loads `libvlc.dll` at runtime (`libloading`), creates a child Win32
window inside the app window, and points libVLC at it with
`libvlc_media_player_set_hwnd`. The child window is repositioned every frame to
match the egui video rectangle. Because that surface is a native child window,
egui cannot paint over it — so the control bar sits directly beneath the video
(and the settings dialog temporarily hides the video while open).

## Layout
| File | Purpose |
|------|---------|
| `src/xtream.rs` | Xtream client: login, categories, items, episodes, URL builders, M3U/EPG |
| `src/video.rs` | libVLC FFI + embedded Win32 child window |
| `src/main.rs` | egui GUI + CLI |

## CLI
```
eglass.exe categories
eglass.exe list live [category_id]
eglass.exe url  live <stream_id>
eglass.exe play live <stream_id>     # CLI play uses an external VLC process
eglass.exe m3u playlist.m3u
eglass.exe epg
```

## Notes
- Blocking HTTP (`ureq`) runs on worker threads; the UI stays responsive.
- Channel switching replaces the stream in the same embedded player.
- Series auto-plays the first episode.
- `max_connections` is enforced by the panel; on many panels concurrent streams from one IP count as one.
