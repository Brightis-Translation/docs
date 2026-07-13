---
title: SCPS_101.05
layout: default
parent: File Specifics
nav_order: 1
---

#### Table of Contents

- [Static Code Analysis](#static-code-analysis)
- [Quick Function Key](#quick-function-key)
- [Triggering the in-game DEBUG MODE](#triggering-debug-mode)

# Static Code Analysis

We've found some functions key in determining how the game works.

- `FUN_80020170`
  - the game's **decompress** function for files.
- `FUN_8007d334`
  - used to **load a rectangle for an image** to be drawn in.
  - The actual function to **load an image**, that uses this, is in`sub_8007d578`.
- `FUN_8001bad8`
  - used to **draw Japanese characters** from the main text box Overlay.
- `FUN_8001bc9c` for **English/ASCII characters** (such as enemy names in the top right)
- `sub_8001F57C`
  - utilizes `DAT_800E0190` which holds an array of 6 elements, which is populated by the file to **handle/track to every BIN file**.
- `sub_8001F57C` **looks up a blob/Overlay part** from one of those 6 BIN files, by a size-offset pair, and return a pointer to that blob.
  - then receives the **index** to the BIN file it should take the blob from (which is indirectly clamped by `sub_8001E12C` due to it loading 6 files)
  - And the `sizeOffsetPair` is size in the lower `0x1FFFF` bits, shifted right by 2. And the top `0x7FFF` bits are the offset into the BIN file.
  - it can be assumed that there ISN'T necessarily a global lookup table to all the blobs in the BIN files, but the game might just hardcode the offset and sizes where it needs it.
  - [ ] Need to find all the references to `sub_8001F57C`, collect the size offset pairs, and later patch them, if any of the blobs changed in size or offset. Maybe by a script or something.
- `sub_8001E208` is used to **load the blobs** from `OVR.BIN`
  - Used this to confirm that the table in `OVR.BIN` encapsulates the whole file.
  - there seems to be a hard cap for `0x3F` sectors in `OVR.BIN`
- `sub_8001e504` **loads textures** with magic identifier `TEX` from either `CHR.BIN` or `MAP.BIN`.
  - these textures type may only be in those two BIN files.

## Quick Function Key

- `FUN_80020170` = decompress
- `FUN_8007d2cc` = DrawSync
- `FUN_8007d234` = SetDispMask
- `FUN_8007cf50` = ResetGraph
- `FUN_8007d0c4` = SetGraphDebug
- `FUN_8007d334` = StoreImage
- `FUN_8007d7b8` = ClearOTagR
- `FUN_8007d1d4` = DrawSyncCallback
- `FUN_8001f74c` = SystemError
- `FUN_800872a4` = GetVideoMode
- `FUN_800976f0` = SpuVoiceAttr
- `FUN_8009b830` = SpuVolume
- `FUN_80099480` = SpuInitHot
- `FUN_8007ba20` = SetGeomOffset
- `FUN_8007ba40` = SetGeomScreen
- `FUN_8009d0c0` = SsSetStereo
- `FUN_8001bc9c` = GetCharacterData
- `FUN_8001bad8` = DrawLargeCharacter
- `FUN_8001d58c` = DrawSmallCharacter
- `FUN_8001cd14` = PrintText (Brightis Specific)
- `FUN_8001cd34` = PrintTextBuffer (Brightis Specific)
- `FUN_8001fb94` = LoadProgOverlay (Brightis Specific)
- `FUN_801540b4` = RenderTextBuffer (Brightis Specific)
- `FUN_80020014` = InitializeOnmOverlayTable (Brightis Specific)
- `FUN_800200ac` = InitializeOverlayTable (Brightis Specific)

Keep an eye on:

- `FUN_8001bad8` - SJIS Controller(?)
- `FUN_8001fb94` - Related to "Ovr Prog" (Prgressive Overlay)
- `FUN_8001fd8c` - Related to "CnstOvr" (Constant Overlay)
- `FUN_8002a4b8` - Mentions of "DEMO PLAY" "DEMO REC" etc.

### Triggering DEBUG MODE

- `DAT_800aebea` has to be `0x800` (or any value that has that bit set at least) (Part of Controler input?)
- `DAT_800aec04` set to `0x20` (or any value that has this bit set)
- `0x801ffffc`(in RAM) must be set to "DeV" or "PrV"

The game has verious checks to see if it's in Debug. We've noticed there are various values it can be set to. All of these Debug modes seem to assume that "no CD (is) inserted."

- `1`: Best for testing and activating the debug mode (will not access Debug Menu 2)
- `2`: Same as above with access to Debug Menu 2 options
- `4`: Testing mode(?)
  - This runs into issues when loading the BIN files actually, if  mode is > `2`, it runs into not only into this trap, but also throws an error when it gets out of it.
  - `4` is also specially checked throughout the game, which might lead to different, non-desirable behaviour.
  - Maybe a mode to load files from an external drive or something?
- `8`: proper debug mode. Makes frequent use of the hardware breakpoint `0x101` from which we don't know how to recover from.  
TODO: Figure out how to get this working outside of Code Patching.

![Debug pg1](https://github.com/user-attachments/assets/9b587dbc-422a-4749-a2a3-e4c43aac628c)
![Debug pg2](https://github.com/user-attachments/assets/1c43ee38-a0db-4a65-9c82-3913b3402607)
