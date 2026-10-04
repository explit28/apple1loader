# Apple-1 Loader ROMs

This repository is a fork of [fstark/apple1loader](https://github.com/fstark/apple1loader), the ROM loader by **Fred Stark** and **Antoine Bercovici** 
for the Aberco / SiliconInsider / Jurassic Computing Apple-1 64K RAM/ROM card.

The fork keeps the original Apple-1 ROM and build system, and also adds a **Briel Replica-1 TE specific 32 KiB ROM image**, additional Apple-1 software, and supporting files used while adapting the loader to the Replica-1 TE memory map.

<img width="738" height="518" alt="Jurassic_64KB_RAM_ROM_Card" src="https://github.com/user-attachments/assets/9fa3ca5e-0e5b-4510-b439-f98589f04247" />
<img width="1072" height="897" alt="Jurassic_64KB_RAM_ROM_Card_Menu" src="https://github.com/user-attachments/assets/74c9608e-74e0-4412-b279-e9adebb7168c" />
<img width="1032" height="1032" alt="Jurassic_64KB_RAM_ROM_Card_Apps" src="https://github.com/user-attachments/assets/a3c032b0-c291-4fac-bce0-0749998df094" />

## ROM images

| File | Target | Start | Status |
| --- | --- | --- | --- |
| `32KA1COMPIL.BIN` | Original Apple-1 + Aberco/SiliconInsider 64K RAM/ROM card | `2000R` | Original/upstream image; rebuilt by the current `Makefile` |
| `32KREPLICA1.BIN` | Briel Replica-1 TE + Jurassic/Aberco-style 64K RAM/ROM card | `9000R` | Replica-1 TE variant; currently provided as a prebuilt 32 KiB image |

> **Important:** the current top-level `Makefile` still builds `32KA1COMPIL.BIN`. It does **not** regenerate `32KREPLICA1.BIN`.

---

# Replica-1 TE ROM

`32KREPLICA1.BIN` is adapted for the **Briel Replica-1 TE**, which differs significantly from a stock Apple-1 memory configuration.

The Replica-1 TE already provides:

- RAM at `$0000-$7FFF`
- PIA / I/O in the `$D000` page
- Integer BASIC at `$E000`
- Krusader at `$F000`
- Woz Monitor at `$FF00`

For that reason the Replica-1 image does not duplicate Integer BASIC, Krusader or WozMon in the external ROM. The loader menu simply jumps to the corresponding built-in entry points.

Start the loader from WozMon with:

```text
9000R
```

## Replica-1 TE menu

The visual layout follows the original Apple-1 Loader menu while keeping the Replica-1 TE specific program set:

```text
   FREDERIC STARK & ANTOINE BERCOVICI
========================================
M) MANDELBROT       1) APPLE 30TH
K) KRUSADER         2) TIC-TAC-TOE
I) INTEGER BASIC    3) LUNAR LANDER
R) BASIC RE-ENTRY   4) LITTLE TOWER
P) 15-PUZZLE        5) C64 MAZE
L) LIFE             6) MICROCHESS
C) DISPLAY TEST     7) PASART
W) WOZMON           8) CELLULAR
E) MINI ASSEMBLER   9) MASTERMIND
?) MEMORY MAP       0) NIM
===================================V1.2=
YOUR CHOICE ->
```

The large `APPLE LOADER` ASCII logo is displayed above the author line.

### Direct Replica-1 TE entries

| Key | Function | Entry point |
| --- | --- | ---: |
| `I` | Integer BASIC | `$E000` |
| `R` | BASIC re-entry / warm entry | `$E2B3` |
| `K` | Krusader | `$F000` |
| `W` | Woz Monitor | `$FF00` |

These entries use software already present in the Replica-1 TE ROM.

### Added / adapted entries

- `P` — **15-Puzzle**, loaded into RAM at `$0300`.
- `L` — **Life**, loaded into RAM at `$6000`.
- `C` — **Display Test** adapted for the Replica-1 TE display path.
- `E` — **Mini Assembler**.
- `?` — **Memory Map** utility adapted to the Replica-1 TE layout. The complete routine includes the CRC code needed when ROM is encountered and returns to the custom loader menu.
- The original numbered applications `1` through `0`, plus `M` Mandelbrot, are retained.

## Why the Apple II Monitor is not in the Replica-1 image

The upstream loader includes an Apple II Monitor entry around `$73F0`. On a Replica-1 TE, `$0000-$7FFF` is already occupied by the machine's internal 32 KiB RAM.

Mapping the external card as ROM in page `7` would therefore overlap the Replica-1 TE RAM and can cause bus contention. The Apple II Monitor was consequently left out of the final Replica-1 TE ROM.

WozMon remains available through `W`, using the Replica-1 TE's built-in monitor at `$FF00`.

## RAM/ROM card mapping for the Replica-1 TE

The Replica-1 image stores loader/program data in the external ROM pages `$8000-$CFFF`.

For the Replica-1 TE configuration used for this ROM:

```text
External card ROM: 8, 9, A, B, C
Do not map:         0-7, D, E, F
```

The reason is:

| Page(s) | Replica-1 TE use |
| --- | --- |
| `$0000-$7FFF` | Internal 32 KiB RAM — do not overlay with the card |
| `$8000-$CFFF` | External loader/program ROM |
| `$D000-$DFFF` | PIA / I/O |
| `$E000-$EFFF` | Built-in Integer BASIC |
| `$F000-$FFFF` | Built-in Krusader / WozMon area |

In particular, **do not enable external ROM page `7`** on a Replica-1 TE.

The `C` page must be available to the ROM for this image. This configuration assumes that no Apple-1 Cassette Interface is occupying that page.

> Jumper labels and ROM/RAM selection details can vary by card revision. Avoid enabling two devices for the same address range.

## Replica-1 TE display detail

The Replica-1 TE character display is 40 columns wide. Writing the 40th printable character already advances to the next row.

For the menu separator lines, the Replica-1 image therefore deliberately does **not** output an additional carriage return after the 40th character. Otherwise an empty row is consumed and the first row of the `APPLE LOADER` ASCII art scrolls off the 24-row display.

This behavior was verified on real Replica-1 TE hardware.

---

# Original Apple-1 ROM

`32KA1COMPIL.BIN` is the original/upstream loader image.

It is built from `32KA1COMPIL.json` and the sources and binaries in `src/`, `software/` and `patches/`.

Start it with:

```text
2000R
```

The original configuration includes the Apple II Monitor, Apple II Mini-Assembler, memory tests, Integer BASIC and WozMon according to the upstream memory map.

For the original ROM's detailed program descriptions and hardware assumptions, see the upstream project:

https://github.com/fstark/apple1loader

Do not use the Replica-1 TE jumper configuration above as a general configuration for the original Apple-1 image; the two ROMs target different memory layouts.

---

# Additional software

The fork also contains additional Apple-1 program images in `software/` for preservation, testing and possible future ROM builds.

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

In particular, some programs were evaluated during development but were intentionally not kept in the final menu. Files that are present in the repository should therefore not automatically be interpreted as contents of either ROM image.

---

# Building the upstream ROM

The existing build system is inherited from the upstream repository.

Requirements include:

- `python3`
- `xa`
- the `cc65` toolchain (`ca65` / `ld65`)
- `wget`
- optionally `minipro` for EEPROM programming

Build the original image with:

```sh
make
```

This creates:

```text
32KA1COMPIL.BIN
```

The EEPROM target is:

```sh
make eeprom
```

The current Makefile programs an `X28C256` with `minipro`.

## Replica-1 image build status

At present, `32KREPLICA1.BIN` is committed as a **prebuilt image**. The current `Makefile` and `32KA1COMPIL.json` describe the upstream `32KA1COMPIL.BIN`, not the Replica-1 TE variant.

Therefore:

```sh
make
```

does **not** reproduce `32KREPLICA1.BIN`.

A future improvement would be to add a separate configuration/build target, for example:

```text
32KREPLICA1.json
make replica1
```

so the Replica-1 image can be rebuilt entirely from source.

---

# Repository layout

```text
.
├── 32KA1COMPIL.BIN      # upstream 32 KiB ROM
├── 32KA1COMPIL.json     # upstream ROM layout/configuration
├── 32KREPLICA1.BIN      # Replica-1 TE 32 KiB ROM
├── Makefile              # currently builds the upstream image
├── makerom.py            # ROM image builder
├── src/                  # assembled source programs / loader code
├── patches/              # patches used by the upstream image
├── software/             # program binaries and added Apple-1 software
├── utils/                # helper tools
└── images/               # screenshots / documentation images
```

---

# Credits

The original loader project, ROM layout and menu were created by **Fred Stark** and **Antoine Bercovici** in 2024.

The ROM collection incorporates historical Apple-1 software from multiple authors. Important credits from the upstream project include:

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
| Lunar Lander | author currently unknown |
| PasArt | author currently unknown |
| Cellular | author currently unknown |
| Nim | author currently unknown |

The Replica-1 TE image is an adaptation of that work for the Briel Replica-1 TE and its different RAM/ROM map.

Please preserve the original authorship information when redistributing binaries or derived ROM images.

## Licensing

This repository contains software from several historical sources and does not currently expose a single root license covering every component. Check the provenance and licensing of individual files before redistribution or reuse.
