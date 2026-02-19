# Hairless MIDI<->Serial Bridge (Undead Edition)

This is a **revival fork** of the original [Hairless MIDI<->Serial Bridge](https://github.com/projectgus/hairless-midiserial) by projectgus, which has been archived and is no longer maintained.

The original project stopped working on modern Windows versions. This fork patches it to compile and run on **Windows 10 and 11** using **Qt 5.15** with MinGW.

**This is not an actively maintained project** — it's a one-time revival to get it working again on modern systems. No new features are planned. If you find a bug, feel free to open an issue or submit a PR, but there are no guarantees of support.

## What is Hairless MIDI<->Serial Bridge?

Hairless MIDI<->Serial Bridge is the easiest way to connect serial devices (like Arduinos) to send and receive MIDI signals.

The original project home page is http://projectgus.github.com/hairless-midiserial/

## What was changed from the original?

- Ported from Qt 4 to Qt 5 (`QT += widgets`, updated deprecated APIs)
- Fixed a critical build bug (wrong source filename in the .pro file for Windows serial port)
- Fixed 64-bit pointer truncation in RtMidi (`DWORD` -> `DWORD_PTR`)
- Replaced non-standard variable-length arrays with fixed-size arrays
- Updated `.gitignore` for modern build artifacts

All underlying Windows APIs (WinMM for MIDI, SetupAPI for serial, overlapped I/O) are unchanged and fully supported on Windows 10/11.

# Building from source

## Prerequisites

Install [MSYS2](https://www.msys2.org/), then open the **MSYS2 UCRT64** terminal and run:

```bash
pacman -S git mingw-w64-ucrt-x86_64-qt5-base mingw-w64-ucrt-x86_64-toolchain
```

## Build

```bash
git clone --recursive https://github.com/chava100f/hairless-midiserial-undead.git
cd hairless-midiserial-undead
qmake hairless-midiserial.pro
mingw32-make
```

The executable will be at `release/hairless-midiserial.exe`.

Run it from the MSYS2 UCRT64 terminal, or copy the required Qt DLLs next to the exe for standalone use.

# Libraries

* [qextserialport](https://code.google.com/p/qextserialport/) — linked into the source tree as a git submodule.

* [The RtMidi library](https://github.com/thestk/rtmidi) — linked into the source tree as a git submodule (with a local patch for 64-bit compatibility).

Both libraries are small so they are compiled as source files directly into Hairless Bridge, not linked as separate libraries.

# Credits

All credit for the original Hairless MIDI<->Serial Bridge goes to [projectgus](https://github.com/projectgus). This fork only applies the minimum changes needed to build on modern systems.
