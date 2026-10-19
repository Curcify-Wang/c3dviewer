# C3DViewer — a Mokka-style C3D / BVH viewer for MATLAB

<p align="center">
  <img src="icon/c3dviewer_icon_256.png" width="128" alt="C3DViewer icon">
</p>

C3DViewer is a MATLAB (uifigure) application that reproduces the workflow and the
mouse/keyboard behaviour of [Mokka](https://biomechanical-toolkit.github.io/mokka/)
on top of the [Moveck Bridge](https://github.com/moveck-community/moveck_bridge_btk)
(BTK-compatible `btk*` API). It opens **C3D**, **TRC** and **Biovision BVH** files,
shows markers, skeletons, force plates and ground-reaction vectors in 3D, plots any
marker/analog/model-output channel, edits events and regions of interest, provides
the usual marker/analog processing tools, and writes the result back to C3D / TRC / CSV.

Author: **Dr. I-Lin Wang** — questions or issues: <ilin@gms.ndhu.edu.tw>

> 中文使用說明請見 [README_c3dviewer.md](README_c3dviewer.md)。

---

## Features

**3D view (Mokka mouse model)**
- Left-drag rotate, middle-drag / Alt+left-drag pan, wheel zoom (anywhere in the view)
- Click to select, Ctrl/Cmd+click to add, Shift+drag box selection, Shift+Ctrl+click to toggle a trajectory
- Selected marker name and X/Y/Z shown inside the view; hover balloons; right-click context menu
- Perspective / Top / Bottom / Front / Back / Left / Right views, 1 / 2 / 4 split layouts
- Floor grid, axis triad, force platforms, GRF vectors and COP
- Marker appearance (colour, radius, visibility, trajectory) and segments (links / faces) with Mokka `.mvc` model files

**Time bar**
- Slider-style time bar with a red current-frame knob and blue ROI handles, event markers (Alt+drag to move)
- Playback at real time, 1/2, 1/4, 1/5, 1/10 or frame-by-frame; loop inside the ROI

**Events and ROI**
- Insert / edit / delete events (table editor, Alt+1–9 shortcuts, right-click on the time bar)
- Gait event detection from force-plate vertical force
- Crop to ROI, reframe from 1, set first frame, resample, convert units

**Marker / analog tools**
- Delete, rename, fill gaps (spline / pchip / linear), Butterworth low/high/band-pass (zero-lag), average marker, distance and angle curves
- Analog: remove offset, filter, rectify, scale, RMS / moving-average envelope, delete — overwrite or create a new `label_suffix` channel

**Charts**
- Multi-series overlay with legend, hover read-out, click-to-seek, wheel zoom on X
- X axis in frames, seconds or **gait-cycle percentage** (cycle mean ± SD)
- Export chart data to CSV

**Import / export**
- Open `.c3d`, `.trc`, `.bvh` (BVH joints become markers, the hierarchy becomes a `Skeleton` segment; auto-scaled to a 1.75 m body height or a chosen unit)
- Save as C3D / TRC; export markers, analogs, model outputs and events to CSV; load/save `.mvc`
- Video synchronisation with a time delay (MATLAB `VideoReader`)

All signal-processing helpers (`c3dv_*.m`) are toolbox-free and also run in GNU Octave.

## Requirements

- MATLAB **R2021a or newer** (developed and tested on R2026a, Apple Silicon)
- [Moveck Bridge](https://github.com/moveck-community/moveck_bridge_btk) for your MATLAB version and platform
  (download it from Moveck; it is **not** included in this repository — check its licence before redistributing it)
- No additional toolboxes. MATLAB Compiler is optional (only for the standalone build).

## Installation

```text
moveck_bridge_btk/
├── moveck_bridge_btk-2025.3.2-matlab2023b-none-macosx_13_0_arm64/   <- Bridge (unzipped)
└── c3dviewer/                                                       <- this repository
```

1. Clone this repository into a folder.
2. Unzip the Moveck Bridge into the **same folder or its parent**. At start-up C3DViewer
   looks for `moveck_bridge_btk-*` next to `C3DViewer.m` (or one level up), picks the newest
   version for your platform and adds it to the path. You can also `addpath` the Bridge yourself.
3. macOS only, first launch: if `btkGetVersion` fails with a Gatekeeper message, run
   `xattr -dr com.apple.quarantine <bridge folder>` once.

## Usage

```matlab
cd c3dviewer
C3DViewer                         % empty window, File > Open...
C3DViewer('trial.c3d')            % open a file; trial.mvc in the same folder is loaded automatically
C3DViewer('walk.bvh')             % BVH: joints -> markers, hierarchy -> Skeleton segment

app = C3DViewer('trial.c3d');     % keep the handle for scripting
app.setFrame(120); app.startPlayback(); app.stopPlayback();
app.setSelection([3 5]); app.loadModelConfig('PluginGait.mvc'); app.saveModelConfig('out.mvc');
```

After updating the source while a window is open, run `C3DViewer.closeAll; clear classes`.

Diagnostics without the GUI: `selftest_C3DViewer('trial.c3d')` (Bridge read test) and
`selftest_bvh('walk.bvh')` (BVH parse + Bridge acquisition test).

## Standalone application (optional)

With MATLAB Compiler installed:

```matlab
cd c3dviewer
build_mcc        % mcc only -> build/C3DViewer/C3DViewer.app (or .exe)
build_app        % compiler.build API -> same app + build/installer (bundles / downloads MATLAB Runtime)
```

The compiled application embeds the Bridge and the icon; its MATLAB code is encrypted inside the
CTF archive. On R2026a the *Application Compiler* GUI may fail with
`Invalid separator input: string` — use the scripts above instead.

## Repository layout

| File | Purpose |
| --- | --- |
| `C3DViewer.m` | the application (single `classdef`) |
| `C3DViewerApp.m` | entry point for the compiled app |
| `c3dv_read_bvh.m` | BVH parser + forward kinematics |
| `c3dv_mvc.m` | Mokka `.mvc` read / write |
| `c3dv_butterworth.m`, `c3dv_fill_gaps.m`, `c3dv_resample.m`, `c3dv_envelope.m`, `c3dv_cycle_normalize.m` | toolbox-free signal processing |
| `c3dv_write_trc.m`, `c3dv_write_csv.m` | exporters |
| `c3dv_file_dialog.m`, `c3dv_param_dialog.m`, `c3dv_segment_dialog.m` | uifigure dialogs |
| `selftest_C3DViewer.m`, `selftest_bvh.m` | command-line diagnostics |
| `build_mcc.m`, `build_app.m` | standalone build scripts |
| `icon/` | application icon (SVG, PNG, ICNS, ICO) |
| `README_c3dviewer.md` | detailed manual (Traditional Chinese) |

## Known limitations

- Band-pass Butterworth is a high-pass + low-pass cascade (gain slightly below 1 at the centre frequency).
- Not implemented from Mokka: STL / image-sequence export, Vicon VSK/VST skeleton import, markerless "Rotations" display.
- `.mvc` attribute names follow Mokka 0.6; parsing is case-insensitive and tolerant of missing attributes.
- Event times are read as `t = (frame - firstFrame) / f` by default; switch in *Settings > Event time reference* if your files use frame 1 = 0 s.

## Acknowledgements

- [Mokka](https://biomechanical-toolkit.github.io/mokka/) and the [Biomechanical ToolKit](https://github.com/Biomechanical-ToolKit) for the interaction model this viewer follows.
- [Moveck](https://www.moveck.com/) for the Bridge that provides the BTK-compatible MATLAB API.

## License

MIT — see [LICENSE](LICENSE). The Moveck Bridge is distributed under its own licence and is not part of this repository.
