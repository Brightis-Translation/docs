---
title: OVR.BIN
layout: default
parent: File Specifics
nav_order: 2
---

#### Table of Contents

- [Pointer Types & Text Offsets](#pointer-types-and-text-offsets)
  - [Command Codes](#command-codes)
  - [Controlling the Text Overlay](#controlling-the-text-overlay)
  - [OVR.BIN Specifics](#OVR.BIN-specifics)
  - [Specific Functions](#specific-functions)
    - [Determining Pointer Tables](#determining-pointer-tables---ovrbin)
    - [Table Slot Assignments](#table-slot-assignments)

# Pointer Types and Text Offsets

After we've gotten a dump of `OVR.BIN`, we started working on listing the decompressed pointers.  
Due to the decompression tool at the time [as of Oct 21st], it is split into 61 parts, by sector, with the trailing zeros removed.  
There ***may*** be issues in some parts, but this is a starting point. Due to this, we've noticed pointers being stored in different methods:

- **Trailing** *[or default/following/etc.]* - following toward the end of the file, or near the top in some cases.
- **Embedded** - Pointers are stuck in the middle of the assembly code.

We're currently using these methods to determine the allignment of pointers to the text:

##### Trailing

```text
`(default RAM starting position) - (pointer value) = (Text starting location)`
```

i.e. `80158180 - 80158138 = 48`  
so our line starts at an offset of 48.

##### Embedded

Since it's much trickier, and seems to be done for Dialogue heavy cutscenes/events. a snippet of code has been graciously provided by esperknight on how to determine the position.

```C#
        if ((PtrValue & 0x8000) >> 15 == 1)
        {
            int ActualPtrValue = (PtrValue & 0xFFFF0000) + ((PtrValue & 0xFFFF) + -0x10000);
            PtrValue += PtrValue - ActualPtrValue;
        }
```

The current WIP allignment of text to pointers can be found at the root of the repo with the title: `Brightis Pointers & WIP translations`

### Command Codes

Refer to the [Text Control Codes](https://brightis-translation.github.io/docs/#text-control-codes) section on the Home page.

### Controlling the Text Overlay

Refer to the [Font Manipulation](https://brightis-translation.github.io/docs/#font-manipulation) section on the Home Page

## OVR.BIN Specifics

(Discovered by Onepiecefreak)

- If you look at the data of OVR.BIN, you will see that it does start out with a table of size-offset pairs, prepended by a 4-byte count.
- Notice how the columns below marked all increase regularly. This wouldn't happen with the compression.  
Those patterns would break up somewhere due to flag bytes having to be in there.
- This pattern acts as a "Table of Contents", and will need to be managed when compressing/decompressing.
  - It tells you how many sectors there are, where they are in the BIN and how big they are.
  - This is not the case for all BIN files and not entirely reliable, but it IS a thing for at least 2 of them. [ONMOVR.BIN & CHR.BIN]
- If you count the 4-byte values from `0x4` onwards, you'll notice, that there are `0x3e` x 4-byte values following it.  
Which makes the value at `0x0` the count for that size-offset table.
- You can also see how the size-offset table pattern breaks after the selected area, another indicator of the end of the table which coincides with the count.
- The same pattern appears in other BIN files
- The size and offsets are shifted, so they can't be taken directly.
- It also seems like the shift logic is different per file, but if you just shift them correctly you WILL be at the start of data.
- We cannot say that table is ALWAYS part of the .BIN file itself, or if this is just part of one of the many parts in that file.
- every overlay >= `044` is a just a table of texts.

Also, it worth noting that after decompressing `OVR.BIN`, code/text related to the debug menu has been found in `019`. It may be useful in controlling text, testing, and more.  
We've since been able to access the menu in game by patching certain address, but it's suspected there is a way to trigger it using a development BIOS.

![image](https://github.com/user-attachments/assets/b2429c0d-7ad1-4d88-bc8f-7ed8b3c6b96f)

### Specific Functions

Functions are split by Overlay into what we'll refer to as "slots."  
These slots will follow the naming convention of: FUN_OVR_`(SLOT #)`_`(Memory Address)`.

- `FUN_OVR_020__80158ec0`
  - the method to look up a character to its glyph, at least in the intro overlay
- `FUN_OVR_020__80159014`
  - surrounds `FUN_OVR_020__80158ec0`
  - loads the lines for the intro sequence, then assumes SJIS (so only ever advances pointers by 2), and looks up the glyphs to render.
    - This explains why ASCII characters can't be easily substituted. As it doesn't have any logic to lookup characters of just 1 byte.
  - When changing the decoding length of one byte, additional issues arise as the function will check for a `NULL` byte.

#### Determining Pointer Tables - OVR.BIN

We believe it could be:

```text
One size-offset pair is split into two 2-byte values, little endian.
Left value is the size, right value is the offset. In the OVR, shift left the offset by 10, shift left the size by 2
```

- Other .BIN files might use other shift values, un-researched currently
- other parts are compressed as well
- loaded by `sub_8001E208` in `SCPS_101.05`
- Confirmed that table IS for the whole BIN, it's part of its structure
- And there seems to be a hard cap for `0x3F` sectors in OVR.BIN

![OVR BIN table](https://github.com/user-attachments/assets/e8e31fa6-09e4-4188-9caf-719532a4d140)

#### Table Slot Assignments

Base Adresses for overlays:

| Type  | Address |
| ------------- | ------------- |
| ProgOverlay | `0x80153eb0` |
| BaseOverlay | `0x80158138` |
| CnstOverlay | `0x8015f328` |
| SubOverlay  | `0x80156CA4` |

All entries are base 0.

`LoadOverlay` calls at addresses:

| Slot | Description | Address |
| ------------- | ------- | ------------- |
| 000 | (Unknown use) | `8002a670` |
| 001 - 010 | (Dungeons 1 - 10) | TBD |
| 012| "Beginner's Dojo" | TBD |
| 014 | ("MOVIE PLAY" for Opening Video file OP00.STR) | `8002cb0c` |
| 014 | ("MOVIE PLAY" for Ending Video file ED00.STR) | `8002cbf4` |
| 015 | Name Entry | TBD |
| 018 | Final Dungeon/Hexarkia Castle | TBD |
| 019 | Debug Menu(s) | `801ffffc` |
| 020 | Intro (Logos + Text Crawl) | `800103b8` |
| 030 | ARC Entertainment & Quintet Credits Sequence | TBD |
| 031 | Signs + Lake Ruins/Inlet Cave Cutscene | TBD |
| 039 | Final Battle | TBD |
| 041 | TECHNIQUE Tutorial | TBD |
| 044 | Inn Wakeup Sequence (post-tutorial Dungeon) | TBD |
| 061 | SCEI Credits Sequence | TBD |