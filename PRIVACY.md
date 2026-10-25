# Privacy

Eagle Glass is a desktop client. It contains no telemetry, no analytics, no
crash reporting and no update check. Nothing is sent to the author.

## What is stored locally

On first run the app creates `%APPDATA%\EagleGlass\config.txt`, containing:

- the Xtream panel server URL,
- the panel username,
- the panel password, in plaintext,
- the preferred live format (`ts` or `m3u8`).

The file is plaintext by design, so treat it like a credential. Use **Reset app…**
in the settings panel to delete it and return to the first-run wizard.

## What leaves the machine

The app makes requests only to the Xtream panel you configure: login,
categories, channel lists, EPG, and the video streams themselves. Those requests
go directly from your machine to your provider. They do not pass through, and
are not visible to, the author of this application.

VLC is loaded from your own local installation of VLC (`libvlc.dll`). This
application does not download, bundle or update it.

## Contact

Open an issue at <https://github.com/gaultpeter/eagle-glass/issues>.
