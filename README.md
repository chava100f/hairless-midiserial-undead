# Hairless MIDI<->Serial Bridge (Undead Edition)

This is a **revival fork** of the original [Hairless MIDI<->Serial Bridge](https://github.com/projectgus/hairless-midiserial) by projectgus, which has been archived and is no longer maintained.

The original project stopped working on modern systems. This fork patches it to compile and run on **Windows 10/11** and **macOS** (Monterey 12+) using **Qt 5.15**.

**This is not an actively maintained project** — it's a one-time revival to get it working again on modern systems. No new features are planned. If you find a bug, feel free to open an issue or submit a PR, but there are no guarantees of support.

## What is Hairless MIDI<->Serial Bridge?

Hairless MIDI<->Serial Bridge is the easiest way to connect serial devices (like Arduinos) to send and receive MIDI signals. It reads MIDI messages from a serial port (USB-to-UART) and forwards them to a virtual MIDI port, so any DAW (Ableton Live, FL Studio, Logic Pro, etc.) can receive them as standard MIDI input.

For more details, see the [original project home page](https://projectgus.github.io/hairless-midiserial/).

## What was changed from the original?

- Ported from Qt 4 to Qt 5 (`QT += widgets`, updated deprecated APIs)
- Fixed a critical build bug (wrong source filename in the .pro file for Windows serial port)
- Updated RtMidi from 2.1.0 to 6.0.0 (with 64-bit Windows fix)
- Fixed deprecated IOKit symbols for macOS 12+ (`kIOMasterPortDefault` -> `kIOMainPortDefault`)
- Removed PowerPC build target, added C++11 support
- Replaced non-standard variable-length arrays with fixed-size arrays
- Replaced git submodules with vendored source files (no more `--recursive` needed)
- Updated `.gitignore` for modern build artifacts

---

# Windows

## Prerequisites

Install [MSYS2](https://www.msys2.org/), then open the **MSYS2 UCRT64** terminal (not regular CMD or PowerShell) and run:

```bash
pacman -S git mingw-w64-ucrt-x86_64-qt5-base mingw-w64-ucrt-x86_64-toolchain
```

## Build

From the **MSYS2 UCRT64** terminal:

```bash
git clone https://github.com/chava100f/hairless-midiserial-undead.git
cd hairless-midiserial-undead
qmake hairless-midiserial.pro
mingw32-make
```

The executable will be at `release/hairless-midiserial.exe`.

## Run

**Option 1 — Run from the MSYS2 UCRT64 terminal:**

```bash
./release/hairless-midiserial.exe
```

This works immediately because MSYS2 adds the Qt DLLs to your PATH.

**Option 2 — Run standalone (double-click from Explorer):**

Copy the required Qt DLLs next to the `.exe`. From the MSYS2 UCRT64 terminal:

```bash
cp /ucrt64/bin/Qt5Core.dll release/
cp /ucrt64/bin/Qt5Gui.dll release/
cp /ucrt64/bin/Qt5Widgets.dll release/
cp /ucrt64/bin/libgcc_s_seh-1.dll release/
cp /ucrt64/bin/libstdc++-6.dll release/
cp /ucrt64/bin/libwinpthread-1.dll release/
mkdir -p release/platforms
cp /ucrt64/share/qt5/plugins/platforms/qwindows.dll release/platforms/
```

Then you can double-click `release/hairless-midiserial.exe` from Windows Explorer.

## MIDI setup (loopMIDI required)

Windows does not have built-in virtual MIDI ports. You need **[loopMIDI](https://www.tobias-erichsen.de/software/loopmidi.html)** (free) to create them.

**How it works:** Hairless Bridge reads MIDI data from your Arduino's serial port, but your DAW (Ableton Live, FL Studio, etc.) can't read serial ports directly — it only understands MIDI ports. loopMIDI creates a virtual MIDI port that acts as the bridge between the two:

```
Arduino (USB serial) → Hairless Bridge → loopMIDI virtual port → Your DAW
```

**Setup:**
1. Install and launch [loopMIDI](https://www.tobias-erichsen.de/software/loopmidi.html)
2. Click the **+** button to create a new virtual MIDI port (give it any name, e.g. "Hairless MIDI")
3. In Hairless Bridge, select the loopMIDI port from the **MIDI Out** dropdown
4. In your DAW, select the same loopMIDI port as a **MIDI Input**
5. Select your Arduino's COM port from the **Serial Port** dropdown in Hairless Bridge
6. Check **"Bridge On"** — MIDI messages from your Arduino will now arrive in your DAW

## Serial drivers

Most Arduino boards (Uno, Nano, Mega) use USB-to-serial chips that Windows 10/11 recognizes automatically. No extra driver installation should be needed — just plug in the Arduino and its COM port will appear in Hairless Bridge.

---

# macOS

## Prerequisites

Install [Homebrew](https://brew.sh/) if you don't have it, then install Qt 5 and Xcode command line tools:

```bash
xcode-select --install
brew install qt@5
```

Add Qt 5 to your PATH (add this to your `~/.zshrc` to make it permanent):

```bash
export PATH="/opt/homebrew/opt/qt@5/bin:$PATH"
```

## Build

```bash
git clone https://github.com/chava100f/hairless-midiserial-undead.git
cd hairless-midiserial-undead
qmake hairless-midiserial.pro
make
```

This creates `hairless-midiserial.app` in the project directory.

## Run

**From the terminal:**

```bash
open hairless-midiserial.app
```

Or double-click `hairless-midiserial.app` in Finder.

## MIDI setup (IAC Driver — built into macOS)

macOS has built-in virtual MIDI support through CoreMIDI, so **no extra software is needed** (unlike Windows). You just need to enable the IAC Driver:

1. Open **Audio MIDI Setup** (search for it in Spotlight, or find it in Applications > Utilities)
2. Go to **Window > Show MIDI Studio**
3. Double-click **IAC Driver**
4. Check **"Device is online"**
5. The "IAC Bus" will now appear as a MIDI port in both Hairless Bridge and your DAW

**Setup in Hairless Bridge:**
1. Select the IAC Bus from the **MIDI Out** dropdown
2. Select your Arduino's serial port from the **Serial Port** dropdown
3. Check **"Bridge On"**
4. In your DAW (Logic Pro, Ableton Live, etc.), select the IAC Bus as a **MIDI Input**

```
Arduino (USB serial) → Hairless Bridge → IAC Bus → Your DAW
```

## Serial drivers

**No FTDI or USB-serial drivers needed on modern macOS.** Apple Silicon Macs (M1/M2/M3/M4) and recent Intel Macs with macOS 12+ include a built-in USB-serial driver (`AppleUSBFTDI`) that handles FTDI, CH340, and CP2102 chips automatically. In older macOS versions (and older Macs with Intel/PowerPC processors), third-party FTDI drivers were required — this is no longer the case.

Just plug in your Arduino and its serial port will appear in Hairless Bridge.

---

# Libraries

* [qextserialport](https://code.google.com/p/qextserialport/) — vendored in `libraries/qextserialport/` (with a local patch for macOS 12+ IOKit compatibility).

* [The RtMidi library](https://github.com/thestk/rtmidi) v6.0.0 — vendored in `libraries/rtmidi/` (with a local patch for 64-bit Windows compatibility).

Both libraries are small so they are compiled as source files directly into Hairless Bridge, not linked as separate libraries. They are included in the repo as regular files (no git submodules), so a plain `git clone` gets everything you need.

# Credits

All credit for the original Hairless MIDI<->Serial Bridge goes to [projectgus](https://github.com/projectgus). This fork only applies the minimum changes needed to build on modern systems.
