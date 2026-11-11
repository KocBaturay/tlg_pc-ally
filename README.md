<p align="center"><img src="port/packaging/icon/tlg-icon-1024.png" width="160" alt="The Last Guardian PC port icon"></p>

<h1 align="center">The Last Guardian PC Port — ROG Ally Fork</h1>

<p align="center">
  <a href="https://github.com/KocBaturay/tlg_pc/releases">Releases</a> ·
  <a href="https://github.com/KocBaturay/tlg_pc/issues">Issues</a> ·
  <a href="https://github.com/KocBaturay/tlg_pc/actions/workflows/build.yml">Builds</a>
</p>

<p align="center"><a href="https://github.com/KocBaturay/tlg_pc/actions/workflows/build.yml"><img src="https://github.com/KocBaturay/tlg_pc/actions/workflows/build.yml/badge.svg" alt="Build"></a></p>

This repository is a fork of [HayatoG's The Last Guardian PC port](https://github.com/HayatoG/tlg_pc), focused on performance and stability for the ASUS ROG Ally.

The game runs natively on the PC processor. The PS4 system is reimplemented, and GPU commands are translated to Vulkan. This fork concentrates on reducing CPU/GPU synchronization stalls and improving frame pacing on handheld hardware.

> **No game files are included.** You need your own copy of The Last Guardian for PS4.

| | |
|---|---|
| ![The Last Guardian main menu](docs/screenshots/01-menu.jpg) | ![The boy wakes in the cave](docs/screenshots/02-despertar.jpg) |
| ![The boy and Trico](docs/screenshots/03-trico.jpg) | ![Ruins](docs/screenshots/04-ruinas.jpg) |

## Performance and stability updates in this fork

- **Eager GPU readbacks:** GPU-written memory windows that the CPU reads are copied to host memory as GPU work is submitted. Later reads can wait for the relevant submission instead of draining the entire GPU queue.
- **Bounded readback tracking:** readback snapshots use 512 KiB windows and 64 tracking slots. Misses are counted and attributed to shader storage, DMA fill/copy, or HLE copy paths to help diagnose performance.
- **Deferred CPU write merging:** eligible CPU write faults on pages written by a shader can proceed without waiting for the whole GPU queue. GPU-owned bytes are preserved while CPU changes are merged back. Windows the CPU has already read are excluded to avoid exposing stale data.
- **Safer draw-stage buffer checks:** draw collection checks mutex-protected pending merge state instead of querying a GPU-modified range set that another stage may update.
- **Stable controller handoff:** the active gamepad no longer alternates between a physical controller and a virtual device that mirrors it.

The readback and write-merge paths are enabled by default. For troubleshooting, `TLG_EAGER_READBACK=0` disables eager readbacks and `TLG_WRITE_MERGE=0` disables deferred write merging. Actual frame rate depends on the game scene, device power profile, and graphics driver.

## Features from the PC port

- **30 FPS by default; experimental 60 FPS and unlocked modes:** choose a frame rate in the launcher or set `TLG_FPS=30`, `TLG_FPS=60`, or `TLG_FPS=unlocked`.
- **Image filters:** SMAA anti-aliasing, RCAS or CAS sharpening, and brightness, contrast, saturation, and gamma adjustments.
- **Mouse and keyboard:** mouse camera control, sensitivity options, inverted Y, and remappable keys and mouse buttons.
- **Motion controls:** DualShock 4 and DualSense gyro support, with a simulated motion mode for other controllers.
- **Japanese text fallback:** experimental text support for the EU CUSA03745 update 01.03; this does not add Japanese-region disc support. See [setup and limitations](docs/japanese.md).
- **PS4-like brightness and lighting:** corrected GPU readback for the game's automatic exposure.
- **Direct memory access for shaders:** optional launcher setting for scenes that need shaders to read game constants directly.
- **Controller options:** stick dead zone and configurable Cross/Circle confirmation. See [menu button settings](docs/menu-buttons.md).
- **Automatic shader cache:** shader cache is stored under `user/cache/`, separate from saves and settings. The launcher can clear this game's compiled shaders.

## Platform and downloads

The ASUS ROG Ally is the primary target for this fork. The inherited port supports 64-bit Windows and Linux with a Vulkan 1.3-capable GPU. Performance varies with hardware, drivers, and the selected power profile.

Builds published for this fork will be listed under [Releases](https://github.com/KocBaturay/tlg_pc/releases). The [Actions page](https://github.com/KocBaturay/tlg_pc/actions) contains automated build runs.

## Required game files

Everything comes from **your own** PS4 (GoldHEN + Itemzflow). Use **The Last Guardian CUSA03627 (US) or CUSA03745 (EU), with update 01.03**, merged into one folder with the base game and update files. The complete installation is about 25 GB.

| File | From | Size | Purpose |
|---|---|---|---|
| `eboot.bin` | update 01.03 | 89 MB | game executable; use the 1.03 version |
| `sce_module/libc.prx`, `sce_module/libSceFios2.prx` | update 01.03 | 1.5 MB | game libraries |
| `sce_sys/` (`param.sfo`, `playgo-chunk.dat`, `icon0.png`, `pic0.png`, `pic1.png`) | base + update | ~5 MB | metadata and launcher artwork |
| `trico_boot.psarc`, `trico_001.psarc`, `trico_002.psarc`, `trico_003.psarc` | base game | 2.8 + 5.8 + 5.4 + 6.8 GB | main content |
| `trico_bootp.psarc`, `trico_001p.psarc`, `trico_002p.psarc`, `trico_003p.psarc` | update 01.03 | ~1.6 GB | update data |
| `trico_enu.psarc`, `trico_ptb.psarc`, `trico_esl.psarc`, `trico_frc.psarc` and update files | base + update | ~280 MB each | voices and text |
| `sce_discmap.plt`, `sce_discmap_patch.plt` | base + update | small | file map; recommended |

The EU release CUSA03745 includes 15 languages. The launcher lists the languages available in the selected game folder.

For native audio, copy `libSceNgs2.sprx` (372 KB) from `/system/common/lib/` on your PS4 into the `system_lib/` folder next to the executable. This file is optional.

To prepare the game folder:

1. Dump the game and update with Itemzflow to an exFAT USB drive.
2. Copy the base game folder to your PC, then copy the update contents over it, replacing files such as `eboot.bin`, `sce_module/`, and `sce_sys/`.
3. Check file sizes. Files larger than 4 GB transferred through GoldHEN 2.2 FTP may be truncated; USB transfer is recommended.

## How to play

1. Extract the package.
2. Optionally place `libSceNgs2.sprx` in `system_lib/`.
3. Open **The Last Guardian** and select the folder containing `eboot.bin`.
4. Choose the language and any available frame rate or image options, then select **Play**.
5. Press **F10** to toggle the FPS counter. Press **Insert** (or R3 + L2 on a controller) to open the port menu.

The launcher supports PS4/PS5, Xbox, and 8BitDo controllers through SDL3, over USB or Bluetooth. Mouse camera controls and remapping are available in the launcher. The full input guide is in [`port/packaging/README-player.md`](port/packaging/README-player.md).

## Troubleshooting and compatibility

- **Incomplete or unusual audio:** without `libSceNgs2.sprx`, the port uses its own experimental mixer, which does not include reverb or filters.
- **Game does not start or closes:** check `out/last_run.log` and attach it to a report in [Issues](https://github.com/KocBaturay/tlg_pc/issues).
- **Older CPUs:** the game uses MOVBE, BMI1, and LZCNT instructions. The port rewrites unsupported instructions at startup; `TLG_ISA_REWRITE=0` disables this workaround.
- **Windows red-zone protection:** the rewrite is enabled by default on Windows. See [`port/docs/red_zone.md`](port/docs/red_zone.md) for details.
- **Windows file errors:** failed file operations are logged with a Win32 error code. Include the log when reporting read or save problems.

## Build from source

Clone this fork with its submodules:

```bash
git clone --recursive https://github.com/KocBaturay/tlg_pc.git
cd tlg_pc/port
TLG_LTO=OFF bash build.sh
```

On Windows, the build uses MSYS2 CLANG64. See [`port/packaging/windows/README.md`](port/packaging/windows/README.md). The workflow at [`.github/workflows/build.yml`](.github/workflows/build.yml) shows the automated Linux and Windows builds.

## Project structure

| Layer | Role | Based on |
|---|---|---|
| Loader | Loads the 1.03 executable and links game and firmware libraries | [bloodborne_pc](https://github.com/deadinside28/bloodborne_pc), [gow3_pc](https://github.com/FelipePrado0/gow3_pc) |
| Runtime | Reimplements PS4 system libraries for threads, files, controllers, audio, and saves | gow3_pc and TLG-specific work |
| Video | Translates Gnm commands and GCN shaders to Vulkan/SPIR-V | [shadPS4](https://github.com/shadps4-emu/shadPS4) |

## Credits and legal

This fork builds on [HayatoG's TLG PC port](https://github.com/HayatoG/tlg_pc), [gow3_pc](https://github.com/FelipePrado0/gow3_pc), [bloodborne_pc](https://github.com/deadinside28/bloodborne_pc), and [shadPS4](https://github.com/shadps4-emu/shadPS4). ATRAC9 audio support uses [LibAtrac9](https://github.com/Thealexbarney/LibAtrac9). See [`port/LICENSE`](port/LICENSE) for the project license.

This project is not affiliated with Sony Interactive Entertainment, SIE Japan Studio, or genDESIGN. The Last Guardian is a trademark of Sony Interactive Entertainment. No game code, files, or artwork are distributed here.
