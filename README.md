# Apple-1 Loader ROMs

ROM images and documentation for the **Apple-1 64K RAM/ROM card** and the **Briel Replica-1 TE**.

This repository is a fork of [fstark/apple1loader](https://github.com/fstark/apple1loader), the ROM loader by **Fred Stark** and **Antoine Bercovici** for the Aberco / SiliconInsider / Jurassic Computing Apple-1 64K RAM/ROM card.

The fork keeps the original Apple-1 ROM and build system and adds a dedicated **Replica-1 TE 32 KiB ROM image**, additional Apple-1 software, and compatibility changes for the Replica-1 TE memory map.

---

## Contents

- [ROM images](#rom-images)
- [Replica-1 TE quick start](#replica-1-te-quick-start)
- [Replica-1 TE hardware configuration](#replica-1-te-hardware-configuration)
- [Replica-1 TE menu](#replica-1-te-menu)
- [Program notes](#program-notes)
- [Replica-1 TE technical notes](#replica-1-te-technical-notes)
- [Original Apple-1 ROM](#original-apple-1-rom)
- [Additional software](#additional-software)
- [Building the ROM](#building-the-rom)
- [Repository layout](#repository-layout)
- [Credits](#credits)
- [Licensing](#licensing)

---

## ROM images

| File | Target | Start command | Status |
| --- | --- | --- | --- |
| `32KA1COMPIL.BIN` | Original Apple-1 + Aberco/SiliconInsider/Jurassic 64K RAM/ROM card | `2000R` | Original/upstream image; rebuilt by the current `Makefile` |
| `32KREPLICA1.BIN` | Briel Replica-1 TE + Jurassic/Aberco-style 64K RAM/ROM card | `9000R` | Replica-1 TE image; tested on real Replica-1 TE hardware |

> **Important:** the current top-level `Makefile` builds `32KA1COMPIL.BIN`. It does **not** currently regenerate `32KREPLICA1.BIN`.

### Current tested Replica-1 TE image

```text
File:    32KREPLICA1.BIN
Size:    32768 bytes
SHA-256: 08136f67ae5db29fcba36da6de2ae00e9d946a8213330a4457ee44b5f436f2ae
```

---

# Replica-1 TE

## Replica-1 TE quick start

### 1. Configure the RAM/ROM card

For the current Replica-1 TE image, use external ROM only in pages:

```text
8 9 A B C
```

Do **not** map the external card as ROM in:

```text
0 1 2 3 4 5 6 7 D E F
```

The most important rule is:

> **Do not enable external ROM page 7 on the Replica-1 TE.**

See [Replica-1 TE hardware configuration](#replica-1-te-hardware-configuration) for the reason.

### 2. Install the ROM

Program `32KREPLICA1.BIN` into the supported 32 KiB ROM/EEPROM used by the 64K RAM/ROM card.

Always power the Apple-1 / Replica-1 off before inserting or removing the card or changing jumpers.

### 3. Start the loader

From Woz Monitor:

```text
9000R
```

The `APPLE LOADER` screen and menu will appear.

---

## Replica-1 TE hardware configuration

### Apple-1 64K RAM/ROM card

The Replica-1 TE ROM in this repository is intended for the Apple-1 64K RAM/ROM card used by the original loader project.

![Apple-1 64K RAM/ROM card overview](images/Jurassic_64KB_RAM_ROM_Card_Technical_Details.jpg)

The card contains a 32 KiB ROM/EEPROM and a 32 KiB RAM device. Its configuration jumpers select, in 4 KiB blocks, whether an address range is provided by ROM, RAM, or neither.

The Briel Replica-1 TE already provides the following memory and firmware:

| Address range / entry | Replica-1 TE function |
| --- | --- |
| `$0000-$7FFF` | Internal 32 KiB RAM |
| `$D000-$DFFF` | PIA / I/O |
| `$E000` | Integer BASIC |
| `$F000` | Krusader |
| `$FF00` | Woz Monitor |

The external RAM/ROM card therefore must not replace these areas.

### 64K RAM/ROM card memory map

The original card documentation shows how the 32 KiB ROM/RAM address space is mapped into the Apple-1 CPU address space:

![Apple-1 64K RAM/ROM card memory map](images/Jurassic_64KB_RAM_ROM_Card_Technical_Details2.jpg)

For the Replica-1 TE we deliberately use only selected external ROM pages, because the Replica-1 already supplies RAM, I/O, Integer BASIC, Krusader, and WozMon in other regions.

### External ROM pages used by this image

```text
External card ROM: 8, 9, A, B, C
Do not map:         0-7, D, E, F
```

| CPU address range | Use |
| --- | --- |
| `$0000-$7FFF` | Replica-1 TE internal RAM — do not overlay |
| `$8000-$CFFF` | External loader/program ROM |
| `$D000-$DFFF` | Replica-1 TE PIA / I/O |
| `$E000-$EFFF` | Built-in Integer BASIC |
| `$F000-$FFFF` | Built-in Krusader / WozMon area |

### Why page 7 must remain disabled

The Replica-1 TE already has RAM at `$7000-$7FFF`.

If external ROM page `7` is enabled at the same time, the external card and the Replica-1 TE RAM can respond to the same addresses. This creates an address-space conflict and can cause bus contention.

For this reason the Replica-1 image deliberately avoids external ROM below `$8000`.

### Page C

The current image uses external ROM page `C`.

This configuration assumes that an Apple-1 Cassette Interface is **not** occupying that address range.

> Jumper labels and ROM/RAM selection details can vary between card revisions. Always verify your card before enabling overlapping address ranges.

---

## Replica-1 TE menu

The visual arrangement follows the original Apple-1 Loader menu as closely as possible while replacing functions that are not appropriate for the Replica-1 TE.

![Apple-1 Loader menu](images/menu%281%29.png)

```text
   FREDERIC STARK & ANTOINE BERCOVICI
========================================
M) MANDELBROT       1) APPLE 30TH
W) WOZMON           2) TIC-TAC-TOE
I) INTEGER BASIC    3) LUNAR LANDER
R) BASIC RE-ENTRY   4) LITTLE TOWER
P) 15-PUZZLE        5) C64 MAZE
L) LIFE             6) MICROCHESS
C) DISPLAY TEST     7) PASART
K) KRUSADER         8) CELLULAR
E) MINI ASSEMBLER   9) MASTERMIND
?) MEMORY MAP       0) NIM
===================================V1.2=
YOUR CHOICE ->
```

The large `APPLE LOADER` ASCII logo is displayed above the author line.

### Relationship to the original menu

The numbered right column remains in the original order:

```text
1  APPLE 30TH
2  TIC-TAC-TOE
3  LUNAR LANDER
4  LITTLE TOWER
5  C64 MAZE
6  MICROCHESS
7  PASART
8  CELLULAR
9  MASTERMIND
0  NIM
```

The original positions of `M`, `W`, `I`, `R`, `C`, `E`, and `?` are also retained.

Only the hardware-specific left-column entries are replaced:

| Original menu row | Replica-1 TE menu |
| --- | --- |
| `A) 8K MEMORY TEST` | `P) 15-PUZZLE` |
| `B) 4K MEMORY TEST` | `L) LIFE` |
| `D) APPLE2 MONITOR` | `K) KRUSADER` |

---

## Program notes

### Built-in Replica-1 TE programs

These menu entries do not duplicate software in the external ROM. They jump directly to software already present in the Replica-1 TE:

| Key | Program | Entry point |
| --- | --- | ---: |
| `W` | Woz Monitor | `$FF00` |
| `I` | Integer BASIC | `$E000` |
| `R` | BASIC re-entry / warm entry | `$E2B3` |
| `K` | Krusader | `$F000` |

### Added or adapted programs

#### `P` — 15-Puzzle

Loaded into RAM at:

```text
$0300
```

#### `L` — Life

Loaded into RAM at:

```text
$6000
```

#### `C` — Display Test

Adapted for the Replica-1 TE display path.

#### `7` — PasArt

PasArt uses the documented Apple-1 memory fix from the
[Apple-1 Software Library](https://apple1software.com/fun/pasart/).

The original byte:

```text
$0308 = $10
```

is changed to:

```text
$0308 = $06
```

This moves PasArt's working-data area from:

```text
$1000
```

to:

```text
$0600
```

The fixed version has been tested successfully on real Replica-1 TE hardware.

#### `E` — Mini Assembler

The Apple II / Apple-1 Mini-Assembler is retained as a separate menu entry.

#### `?` — Memory Map

The Memory Map utility is adapted to the Replica-1 TE layout.

The complete routine includes the CRC code needed when ROM is encountered and returns to the custom loader menu when finished.

### Original numbered applications

The original numbered applications remain in the ROM:

- `1` — Apple 30th Anniversary Demo
- `2` — Tic-Tac-Toe
- `3` — Lunar Lander
- `4` — Little Tower
- `5` — C64 Maze
- `6` — Micro-Chess
- `7` — PasArt
- `8` — Cellular
- `9` — Mastermind
- `0` — Nim

`M) Mandelbrot` is also retained.

---

## Replica-1 TE technical notes

### Why the Apple II Monitor is not included

The upstream loader includes an Apple II Monitor in the `$7xxx` address range.

On a Replica-1 TE, `$0000-$7FFF` is already internal RAM. Mapping external ROM into page `7` would overlap that RAM.

The Apple II Monitor is therefore not included in the current Replica-1 TE image.

Its original menu row is used by:

```text
K) KRUSADER
```

which jumps directly to the Replica-1 TE's built-in Krusader at `$F000`.

Woz Monitor remains available through:

```text
W) WOZMON
```

using the built-in WozMon at `$FF00`.

### 40-column display behavior

The Replica-1 TE character display is 40 columns wide.

Writing the 40th printable character already advances the cursor to the next row. Therefore the Replica-1 menu deliberately does **not** send an additional carriage return after its 40-character separator lines.

Without this change, an empty display row would be consumed and the first row of the `APPLE LOADER` ASCII logo could scroll off the 24-row screen.

This behavior has been verified on real Replica-1 TE hardware.

---

# Original Apple-1 ROM

## `32KA1COMPIL.BIN`

`32KA1COMPIL.BIN` is the original/upstream Apple-1 loader image.

It is built from:

- `32KA1COMPIL.json`
- sources in `src/`
- program binaries in `software/`
- patches in `patches/`

Start it from WozMon with:

```text
2000R
```

The original configuration includes functions such as:

- Apple II Monitor
- Apple II Mini-Assembler
- memory tests
- Integer BASIC
- Woz Monitor
- the original loader applications

For the original ROM's detailed memory layout and hardware assumptions, see the upstream project:

[https://github.com/fstark/apple1loader](https://github.com/fstark/apple1loader)

> **Do not use the Replica-1 TE jumper configuration as a general configuration for the original Apple-1 image.** The two images target different memory maps.

---

## Additional software

The fork also contains additional Apple-1 program images in `software/` for preservation, testing, and possible future ROM builds.

Examples include:

```text
15-puzzle
life
aceyducey
adventure
bowling
buzzword
codebreaker
craps
deal
hammurabi
hundred
slots
startrek
wumpus
```

Not every file in `software/` is included in `32KREPLICA1.BIN`.

Some programs were evaluated during development but intentionally not kept in the final menu. The presence of a file in `software/` therefore does not automatically mean it is included in either ROM image.

---

# Building the ROM

## Requirements

The inherited upstream build system uses:

- `python3`
- `xa`
- `cc65` toolchain (`ca65` / `ld65`)
- `wget`
- optionally `minipro` for EEPROM programming

## Build the original Apple-1 image

```sh
make
```

This creates:

```text
32KA1COMPIL.BIN
```

## Program the EEPROM

```sh
make eeprom
```

The current EEPROM target uses `minipro` and is configured for an `X28C256`.

## Replica-1 TE build status

`32KREPLICA1.BIN` is currently committed as a **prebuilt 32 KiB image**.

The current:

```text
Makefile
32KA1COMPIL.json
```

describe the upstream `32KA1COMPIL.BIN`, not the Replica-1 TE image.

Therefore:

```sh
make
```

does **not** reproduce `32KREPLICA1.BIN`.

A future improvement would be a separate Replica-1 configuration and build target, for example:

```text
32KREPLICA1.json
make replica1
```

---

# Repository layout

```text
.
├── 32KA1COMPIL.BIN      # original/upstream 32 KiB ROM
├── 32KA1COMPIL.json     # upstream ROM layout/configuration
├── 32KREPLICA1.BIN      # Replica-1 TE 32 KiB ROM
├── Makefile             # currently builds the upstream ROM
├── makerom.py           # ROM image builder
├── src/                 # assembled source programs / loader code
├── patches/             # upstream patches
├── software/            # Apple-1 program binaries
├── utils/               # helper tools
└── images/              # screenshots and documentation images
```

---

# Credits

The original loader project, ROM layout, and menu were created by **Fred Stark** and **Antoine Bercovici** in 2024.

This repository is derived from:

[https://github.com/fstark/apple1loader](https://github.com/fstark/apple1loader)

The ROM collection incorporates historical Apple-1 software from multiple authors.

| Software | Author / credit |
| --- | --- |
| WozMon | Steve Wozniak, 1976 |
| Integer BASIC | Steve Wozniak, 1976 |
| Apple 30th Anniversary Demo | Dave Schmenk, 2006 |
| Tic-Tac-Toe | Larry Nelson, 1977 |
| Little Tower | Arnaud Verhille, 2000; fixes by Fred Stark, 2024 |
| C64 Maze | Antoine Bercovici, 2024 |
| Micro-Chess | Peter R. Jennings, 1976 |
| Mini-Assembler | Allen Baum, 1976 |
| Mandelbrot | Fred Stark, 2024 |
| Memory Map | Fred Stark, 2024 |
| PasArt | Ken Wesson, 2007 |
| Lunar Lander | author currently unknown |
| Cellular | author currently unknown |
| Nim | author currently unknown |

The Replica-1 TE image adapts this work to the Briel Replica-1 TE and its different RAM/ROM map.

Please preserve the original authorship information when redistributing binaries or derived ROM images.

---

# Licensing

This repository contains software from several historical sources and does not currently expose a single root license covering every component.

Check the provenance and licensing of individual files before redistribution or reuse.
