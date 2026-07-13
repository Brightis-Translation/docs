---
title: ONMOVR.BIN
layout: default
parent: File Specifics
nav_order: 3
---

#### Table of Contents

- [What this file does](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/ONMOVR.BIN#what-this-file-does)
- [Overlays](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/ONMOVR.BIN#overlays)
- [Determining the pointer table structure](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/ONMOVR.BIN#determining-the-pointer-table-structure---onmovrbin)
- [Overlay Addresses & Slot Assignments](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/ONMOVR.BIN#overlay-addresses-and-slot-assignments)

#### What this file does
From our assumptions, the file `ONMOVR.BIN` is used for anything related to the player's data stats, equipment and more.  
This includes:

- Shop items [Armor, Weapons, Items]
- Skills/Techniques
- Gold/XP
- Any "Pause" menu items [Controller/Camera Settings]
  
# Overlays

There are three(3) main overlay files used at any given time in game.  
We'll call them: `LoadOverlay`, `LoadCnstOverlay`, & `LoadProgOverlay`.

Their respective names correlate to the addresses in memory listed below.  
That means `LoadCnstOverlay` loads an overlay from `ONMOVR.BIN` to the `CnstOverlayAddress`.

Given that, only three(3) overlays can be active at the same time (at my current knowledge). One(1) from `OVR.BIN`, two(2) from `ONMOVR.BIN`.

Given the base addresses, the size a single overlay can have is limited, but that shouldn't be an issue.  
It should be mentioned that findings so far are explicitly calls to load overlays from the main executable (`SCPS_101.05`).

Overlays can load other overlays, given they don't interefere.  
Overlay-to-overlay loads have not been researched yet.

For certain scenarios, the main Overlay Drawing method can be copied into `ONMOVR_002` (since pretty much all cutscenes in the dungeons use it).

- it doesn't patch the method everywhere, and since other text patching methods are already in ONMOVR_002, it fits the purpose
- Scenarios include some menus and certain dialogs (popup text w/o the colored overlay) from the main executable, which means we'll have a mix of variable width font in ASCII, and Full Width

![image](https://github.com/user-attachments/assets/e3512539-c1a3-4bd3-93fa-09a3f0f9710d)

## Determining the Pointer Table Structure - ONMOVR.BIN

We believe it could be:

- Following the same pattern as `OVR.BIN` as it:
  - follows the mentioned pattern
  - Count checks out
  - table pattern visibly stops after selected area
  - columns only contain increasing values in an unbroken pattern
- The offset should only be shifted by two `2`, the size also shifted by `2`.
- This one MAY be the table for the whole BIN file.

![ONMOVR BIN table](https://github.com/user-attachments/assets/2f07a2bc-dbae-4eb4-8822-3e7cc2b21021)

## Overlay Addresses and Slot Assignments

Base Adresses for overlays:

| Type  | Address |
| ------------- | ------------- |
| ProgOverlay | `0x80153eb0` |
| BaseOverlay | `0x80158138` |
| CnstOverlay | `0x8015f328` |
| SubOverlay  | `0x80156CA4` |

All entries are base `0`.

`LoadCnstOverlay` calls at addresses:

- `8001c91c` -> Slot 016
- `8002a5dc` -> Slot 016
- `8003c794` -> Slot 016
- `8003270c` -> Slot 018
- `8007af40` -> Slot 027
- `8002a588` -> Slot 027

`LoadProgOverlay` calls at addresses:

- `8002c708` -> Slot 000
- `8007ae24` -> Slot 001
- `8001cd4c` -> Slot 002 ("DEMO PLAY")
- `8003f950` -> Slot 003 ("DEMO PLAY")
- `8001cd7c` -> Slot 019
- `8003d44c` -> Slot 019
- `8002a4f0` -> Slot 028
- `8002a6ec` -> Slot 028
- `8002a70c` -> Slot 028
