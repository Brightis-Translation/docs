---
title: Home & General Discoveries
layout: home
nav_order: 1
permalink: /
---

# Welcome to the Brightis Translation Wiki

If you would like to help, or follow along with our process, please check the [Getting Started](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/Getting%20Started) doc.

This repo will have all the latest changes & discoveries. Please disregard the original General Discoveries file.

{{< /*<!-- markdownlint-disable MD001 MD022-->*/ >}}
#### Table of Contents

- [General Text Discoveries](#general-text-discoveries)
  - [Other Text Details](#other-text-details)
  - [Font Manipulation](#font-manipulation)
  - [Addtional readings on how Overlays on PSX](#addtional-readings-on-how-overlays-on-psx-similarly-to-brightis)
  - [Text Control Codes](#text-control-codes)
- [General Image Discoveries](#general-image-discoveries)
- [Decompression Notes (& what to look out for)](#general-image-discoveries)
- [Ghidra Discoveries](#ghidra-discoveries)

## General Text Discoveries

ALL text that we have discovered gets loaded via an overlay.  
There are 4 types, each with their own location in RAM where text gets loaded into.

| Overlay Type  | value | Description |
| ------------- | ------------- | -------- |
| Base Overlay | `80158138` | The default overlay, used in MOST cases onscreen |
| Prog(ressive) Overlay | `80153EB0` | displays text, then auto-advance by scrolling until a control code instructs a stop. |
| Cnst(Constant) Overlay | `8015F328` | Used in menus & shops that will stay static. |
| Sub Overlay | `80156CA4` | used w/ other overlays (such as Base or Cnst) to make additions/varibles show as needed. |

### Other Text Details

- Text is (mostly) readable using a Shift-JIS table once decompressed.
- Most in-game text is displayed using a BIOS standard font (due to the regular calls to the `Krom2RawAdd` PSX library function.)
- Some files seem to be normally compressed in `OVR.BIN` while other text are compressed using different methods.
- Pointers are used differently across files.
  - ONMOVR.BIN appears to use Immediate Pointers.
- The locations at which the overlays are put in RAM are static. If one overlay is too big, it would interfere with another overlay.
- Any ASCII code displayed in game is converted to SJIS Full Width via subroutines.
- Every character displayed is 4px wide, and clips into the character itself when reduced.
- The game respects the `RECT` used to draw the overlay, and will not draw outside it.
- The frame supports about 22 Full-Width SJIS characters per line, with 3 lines max.

![First text patch](https://github.com/user-attachments/assets/1deab96c-d06d-49d2-8ec1-cbad1096ace8)

### Font manipulation

Need to lock down where this is controlled in code:

- `8001c0c0` - Set Character Widths
- `8001c0c4` - Set Character Height (from top)
- `8001c0c8` - Set Character Height (from bottom)
- `8001c0d0` - Set Font total height (will squish character)
- `8001c09d` / `8001c070` / `8001c060` - Set Character boldness
- `8001c09c` - Set Character Aliasing
- `8001c06c`  / `8001c05c` - Set Character Transparency
- `8001c0b8` - Character kerning

### Addtional readings on how Overlays on PSX [similarly to Brightis]

- [Working code example](https://github.com/JaberwockySeamonstah/PSXOverlayExample)
- [proper write up going into detail](https://www.beneaththewaves.net/Software/TDR_Practice_Using_OVERLAYS.html#WalkthroughOVERLAYSOverlays)

### Text Control Codes

All control codes are 1 byte (less than `0x20`) & examples can be found wherever there is SJIS text (such as `002_ONMOVR`) at `0x8015463c` in RAM.

| Code  | Action |
| ------------- | ------------- |
| 4  | Stops text, req. button press, starts at the top left in new textbox |
| 5  | Stops text, req. button press, continues same textbox  |
| 7  | Creates linebreak  |
| 10  | represents player name to be inserted |
| 11(dec) / 0B(hex) | Denotes a wait time in frames in the following bit. (typically 15) |
| 14, 17| Enable "jumping" in text to certain lines.  14 jumps unconditionally, 17 will use the flags 16/19. Also uses labels (1/2/3). |
| 15(dec) / 0F(hex) | Typically found as a wait time in frames |
| 16 | Works with 14/17, acting as a jump location in the overlay |
| 19 | Works with 14/17, setting a flag for a "jump." |
| 18, 21, 25 - 31  | Invalid codes/skipped by Overlay  |

## General Image Discoveries

- Palette data appears to be stored in the `SCPS_101.05`
- Game uses Standard TIM matches.
- found many Magic Tag hex sequences
  - (A magic tag "`0×10000000`" is basically what Identifies if it is an image or not)

We've recently managed to extract how text glyphs are used in game

- The game takes the BIOS font and in-code actually modify it to have various styles
- The BIOS font contains 16x15 glyphs, each bit just describes if a pixel should be set or not. And in one instance they make it a 16x16 glyph (bottom line is always transparent) and the colors and the drop shadow are added when interpreting the bits of the BIOS font.
- The glyphs go through a different routine when they are used for text box text.
- The devs create new glyphs from the BIOS font on the fly

![image](https://github.com/user-attachments/assets/f43a0468-c54c-4f06-9f3b-affcf51d9ada)

## Decompression Notes (& what to look out for)

(Discovered by Onepiecefreak)

- If the data is coincidentally set up in a way that displacements never go out of the start of the buffer, you can decompress any buffer given to it.
- For repetition you only need 2 bytes along with a properly set up bit pattern in the flag for it.  
So you most likely end up with either a seqeuence of raws until you hit the offset, or sliding window pairs that don't go back too much by coincidence
- Just being able to be decompressed is never a sure-fire way to declare something compressed
- Most .BINs use a "table of contents" at the start to track where data is located across blobs/sectors. These need to be updated as changes are made to the files.
  - Within entry/slot, specific items can be listed to load into memory. They are limited, so any additions will require patching.

- The game has a common buffer to store something it currently has to process. Be it image data, text objects, or whatever. If it's currently needed, the game puts it at `0x801beee8`
  - Issue: when loading an overlay (and by extension any file that needs decompression) also uses this location.
  - i.e. if you inject an overlay in places the game initially didn't do it, we run the risk of overwriting overlay memory & the common buffer, when the game is using it.
    - Example: When exiting the save/load menu using the newly patched draw method (which injected loading a different overlay for all the ASCII related patching necessary) the common buffer was overwritten, while the save/load menu was actively using that point to keep track of which texts to print.
  - Therefore, it's very possible that we can't patch this draw method to use ASCII.

## Ghidra Discoveries

Using Ghidra, I was able to find some comments at the top of some files & functions. They may be useful so I'll save them here.

 ```text
 SYSTEM.CNF: // ram:00000000-ram:0000003c ()
 OVR.BIN: // ram:00000000-ram:000d8fff 
 PDADOC.BIN: //ram:00000000-ram:0019e7ff 
 PDADOWN.BIN // ram:00000000-ram:000d8fff 
 SCPS_101.05 [Has the main() function, mutlitple ram locations]
   - main // ram:80010000-ram:8009e30f 
   - cache // ram:1f800000-ram:1f8003ff 
 SND.BIN // ram:00000000-ram:0044ffff 
 ONMOVR.BIN // ram:00000000-ram:0001279b 
 CHR.BIN // ram:00000000-ram:0039a7ff 
 MAP.BIN // ram:00000000-ram:010d9fff 
 PDADOWN.EXE [Another main() function]
   - main // ram:80010000-ram:80037dbf 
   - cache // ram:1f800000-ram:1f8003ff 
 ```

These functions can be found using the decompilations in the `Decrypted Files/Ghidra` folder, and browsing them in Ghidra or IDA.
