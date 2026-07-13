---
title: CHR.BIN
layout: default
parent: File Specifics
nav_order: 4
---

#### Table of Contents

- [CHR.BIN Specifics](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/CHR.BIN#CHR.BIN-Specifics)
- [Determining Pointer Tables](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/CHR.BIN#Determining-Pointer-Tables---CHR.BIN)
- [Table Slot Assignments](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/CHR.BIN#table-slot-assignments)

# CHR.BIN Specifics

- Lookups
  - Offset has to be shifted to the left by 11 (or multiply by `0x800`)
  - Size has to be shifted left by 2 (or multiplied by 4)
- These are continuous files. Between them are `00` patches only [Could be used to map out the rest of `CHR.BIN` if consistent]
- The value of the offset is the address stored.
  - If you ever have to edit those files (I think they are mostly textures) you would have to edit the overlay (i.e. slot `020`) at those addresses.

## Determining Pointer Tables - CHR.BIN

We believe it could be:

- Following the same pattern as `OVR.BIN` as it:
  - follows the mentioned pattern
  - Count checks out
  - table pattern visibly stops after selected area
  - columns only contain increasing values in an unbroken pattern
- The offset should only be shifted by 2, the size also shifted by 2.
- This one may not be the table for the whole BIN file, but maybe just one blob of many.
- Shifting the offset here doesn't make much sense beyond 2, and never coincides with clear `00` patches.

![CHR BIN table](https://github.com/user-attachments/assets/7febfb9f-197e-49a0-b4f9-0b27d87fdfee)

### Table Slot Assignments

Base Adresses for overlays:

| Type  | Address |
| ----------- | ------------ |
| ProgOverlay | `0x80153eb0` |
| BaseOverlay | `0x80158138` |
| CnstOverlay | `0x8015f328` |
| SubOverlay  | `0x80156CA4` |

All entries are base 0.  
`LoadOverlay` calls at addresses:

| Type  | Address |
| -------- | -------- |
| Slot 000 | "DEMO PLAY" Overlay |
| Slot 002 | "DEMO PLAY" Overlay |
| Slot 020 | Intro text crawl |
