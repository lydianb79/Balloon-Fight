# Balloon Fight — native macOS static recompilation

This folder turns your `Balloon Fight.nes` into a native Mac app. Nothing is
emulated instruction-by-instruction: the game's 6502 machine code is
translated ahead of time into C and compiled by clang into an arm64 + x86_64
binary.

## Build

Double-click **build.command** (first time: right-click → Open). It needs
Apple's free Command Line Tools and offers to install them if they're missing.
The result is `Balloon Fight.app`, next to your ROM.

To use a ROM somewhere else: `./build.command /path/to/rom.nes`

## Controls

Defaults (change them in **Settings → Controls**, ⌘,):

| | Player 1 | Player 2 |
|---|---|---|
| Move | Arrow keys | W A S D |
| A (flap) | X | H |
| B (rapid flap) | Z | G |
| Start / Select | Return / Tab | — |

Each action takes one key and one controller button per player. Controller 1
drives Player 1, controller 2 drives Player 2, and the left stick always steers.
Bindings are saved automatically.

## Video

**Settings → Video** or the **View** menu:

* **Filtering:** Nearest (sharp pixels), Linear (smooth), or Sharp linear
  (whole-pixel nearest scaling first, then linear smoothing for the remainder).
* **Pixel aspect:** square, or NTSC TV 8:7.
* **Scanlines:** Off, Light, Medium or Strong; a dark band between game lines,
  shown once the picture is at least 2× tall. Most even with integer scaling.
* **Integer scaling** and **Crop overscan** (hides the top and bottom 8 lines).

⌘R reset · ⌘P pause · ⌘1–4 window size · ⌃⌘F full screen.

## How it works

* `recomp.py` traces all code reachable from the reset/NMI/IRQ vectors (plus
  `entries.txt`, the targets of the game's one indirect `JMP ($0025)`, found by
  playtesting) and emits one C label per 6502 instruction. Branches, jumps and
  calls become `goto`s; returns go through a dispatcher. The 6502 stack is
  kept in emulated RAM, so stack tricks behave like on hardware. Code is split
  into ~160-instruction functions to keep compile times short.
* Anything the trace missed falls back to a generated interpreter with the
  same semantics. In 360,000 frames of randomized playtesting across 1P, 2P and
  Balloon Trip, the fallback was never hit.
* `runtime/` holds the hardware: PPU (scanline renderer with dot-timed vblank,
  NMI and sprite-0 hit), APU (2 pulse, triangle, noise, DMC with the NES
  mixer), controllers. `platform/mac/main.m` is the Cocoa/Core Animation/Core
  Audio/GameController front-end.
* The generated C (`gen/`) and the app are built from your ROM on your Mac
  only; this folder contains no game code or data.
