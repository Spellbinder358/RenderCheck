# RenderCheck

**Checks that every camera original has a matching transcode, and that the frame counts agree.**

RenderCheck is a small Mac menu bar app for DITs and post teams. Point it at your camera originals and your transcodes, and it tells you what is missing, what is extra, and which clips don't have the same number of frames.

- Compares clip names between the two folders (rules built for ARRI, RED, Sony, Blackmagic, Phantom, DJI and Avid workflows)
- Optionally compares **exact frame counts** for every clip
- Handles conformed frame rates (for example 240 fps footage conformed to 23.976) with a second check
- Saves a full text report after every run, and exports a PDF
- Lives in the menu bar, with no Dock icon, and can open at login
- Nothing leaves your computer. RenderCheck doesn't connect to the internet.

---

## Install

**Requires macOS 13 (Ventura) or later.** Apple Silicon and Intel are both supported when the app is built as a universal app.

1. Download the latest `RenderCheck-x.xx.zip` from the [Releases](../../releases) page.
2. Unzip it and drag **RenderCheck** into your **Applications** folder.
3. The first time, **right-click the app and choose Open**, then click **Open** again. (The app is not notarized by Apple, so macOS asks once.)
   If macOS says the app is "damaged", run this in Terminal and try again:
   ```
   xattr -cr /Applications/RenderCheck.app
   ```
4. Keep it in **Applications** so *Open at login* works reliably.

MediaInfo, which reads frame counts, is built into the app. Nothing else needs installing, except REDline if you want RED frame counts (see below).

---

## Using it

RenderCheck sits in the menu bar (the checkmark icon):

| Menu item | What it does |
|---|---|
| **Check Now** | Opens the window and runs a check with the folders from last time |
| **Open Interface** | Opens the window |
| **Settings…** | Opens Settings |
| **Quit RenderCheck** | Quits the app |

In the window:

1. Drop your **Camera Originals** folder on the top box, or click **+** to choose it. The folder chooser for the second box opens in the same place as the first, so both usually come from the same drive.
2. Do the same for **Transcodes** below it.
3. Choose **Names + Frame Counts** or **Names Only**.
4. Click **Check**.

While frame counts are running you can click **Skip Frame Count** to fall back to a names-only check.

### Results

- **Missing**: originals with no transcode
- **Extra**: transcodes with no original
- **Frame counts**: clips where the frame counts differ, or a file couldn't be read
- **Not checked**: clips that can't be frame-counted
- **Notes**: for example, a tool that wasn't found

**Save Report…** saves the text report, and **Export PDF…** makes a PDF. The latest report is also kept at `~/Library/RenderCheck/report.txt`.

The window shows only what you need to act on. The report adds the detail: percentages, durations, and any clip that passed the second check.

---

## How clips are matched

Originals and transcodes are compared by **clip name**, not by file name:

- The extension is removed.
- Names are cut to their **first 10 characters**, except names containing `Cine`, `Blackmagic` or a year pattern (`_2013`, `_2014-`, `_2015-`, `_2016`, `_2017-`, `_2018`, `_2019`, `_2020-`), which keep their full length.
- A trailing `_`, or `_` plus one character, is removed (so RED `A001_C002_1` becomes `A001_C002`).
- ARRIRAW frame sequences (`.ari`, `.arx`) are treated as one clip, using the first 20 characters of the name.
- DJI DNG sequences count once per clip, using the first frame.
- RED `.RDC` folders count as one clip; the R3D files inside them are not listed separately.
- Avid's extra audio files (`_A1.mxf`, `_A01.mxf` …) and `.wav` files are ignored.
- If your transcodes folder sits inside your originals folder, it is skipped when reading the originals.

**Originals:** `.mov`, `.mp4`, `.mxf`, `.RDC`, `.cine`, `.CRM`, `.ari`, `.arx`, Blackmagic and DNG.
**Transcodes:** `.mov` and `.mxf` (lowercase extensions).

---

## Frame counts

Frame counts must match **exactly**. Where each number comes from:

| Source | Used for |
|---|---|
| **MediaInfo** (built in) | MOV, MP4, MXF originals and transcodes |
| **REDline** | RED `.RDC` clips |
| **File count** | ARRIRAW `.ari` / `.arx` sequences (one file per frame) |
| **Sony XML sidecar** | Sony Venice / Burano MXF originals that have a `…M01.XML` next to the clip. Much faster than reading the MXF |
| **ALE file** | Originals that have an Avid ALE file in the clip's folder, or up to three folders above. Duration is converted to frames |

CRM, cine and DNG originals are listed under **Not checked**.

The sidecar and ALE numbers are written by the camera, not read from the media. They would not reveal a damaged original. You can switch either off in Settings.

**Second check.** If a clip's frame counts differ, RenderCheck compares durations and frame counts. If either agrees within **0.15%**, the clip passes and appears only in the report. This covers conformed frame rates, such as 239.76 fps to 240 fps or 240 fps footage conformed to 23.976.

### RED clips

RED frame counts need **REDline**, which comes with REDCINE-X PRO. RenderCheck looks for it automatically. If it can't find it, choose its location in **Settings**. REDline can't be bundled with the app.

---

## Settings

Open from the menu bar icon or the gear in the window.

- **Open RenderCheck at login**
- **Start with the folders from last time**
- **Files read at the same time**
- **Quick MediaInfo read**
- **Use Sony XML files** and **Use ALE files**
- **REDline location**
- **Version**

---

## Build it yourself

You need the Xcode command line tools (`xcode-select --install`) and the MediaInfo **command-line** tool for macOS from <https://mediaarea.net/en/MediaInfo/Download/Mac_OS>.

1. Put the `mediainfo` file next to `build_app.sh` (or install it on your Mac).
2. Run:
   ```
   chmod +x build_app.sh
   UNIVERSAL=1 ./build_app.sh
   ```
   Leave off `UNIVERSAL=1` to build only for your own Mac.
3. The app is `build/RenderCheck.app`, and a shareable `build/RenderCheck-x.xx.zip` is created next to it.

The version number comes from `Resources/VERSION`.

To publish a new version: bump `Resources/VERSION`, build, then create a new **Release** on GitHub (tag `vX.XX`) and attach the zip from `build/`.

---

## Troubleshooting

- **"MediaInfo was not found" when building.** Put a file named exactly `mediainfo` in the project folder, or run `./build_app.sh /path/to/mediainfo`.
- **macOS asks to access a removable drive or network volume.** Click Allow. RenderCheck only reads the folders you choose.
- **Open at login doesn't stick.** Move the app into **Applications**, then turn the setting off and on again.
- **A RED clip shows as not read.** REDline wasn't found. Set its location in Settings.
- **Frame counting is slow on some originals.** Raise *Files read at the same time* in Settings. Sony and ALE sidecars skip the slow reads where they exist.

---

## Credits

Frame counts are read with [MediaInfo](https://mediaarea.net/en/MediaInfo) by MediaArea.net SARL, included under its license (<https://mediaarea.net/MediaInfo/License>).
