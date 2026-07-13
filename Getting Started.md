---
title: Getting Started
layout: default
parent: General Info & Common Notes
nav order: 1
---

# Welcome!

Let me first thank you for your interest in getting involved in the project!  
The current focus for us is: **Gathering Pointers for Text in the game.**

We plan on documenting how we did things here in the future, and what the tools do in case others wish to translate the game into other languages, or use our methods for other games!

##### Table of Contents

- [Required Tools & Setup](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/Getting-Started#required-tools--setup)
  - [Decompressing the Text](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/Getting-Started#Decompressing-the-Text)
  - [Getting the Pointers](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/Getting-Started#Getting-the-Pointers)
  - [Patching in the Text](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/Getting-Started#Patching-in-the-Text)
  - [Patching the Game](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/Getting-Started#Patching-the-Game)
  - [Building a Reusable Patch - Ideas](https://github.com/Dickdebonair/Brightis-fan-translation/wiki/Getting-Started#building-a-resuable-patch---ideas)

### Required Tools & Setup

While the README at the root of the repo has the majority of tools we are using, some tools are more necessary than others.  
The bare minimum to complete what we've done so far are:

- [CD Mage](https://www.romhacking.net/utilities/1435/) - tool used to open/insert the game files. This works best with game(s) in the `.BIN/.CUE` format. Open the `.CUE` file in the program.
- [Ghidra](https://ghidra-sre.org) - One of the best Reverse Engineering tools that's completely free. It will be used to decompile and make sense/changes to the game's code as needed.
- [Ghidra: PSX Loader](https://github.com/lab313ru/ghidra_psx_ldr) - This is an extention that will allow us to analyze files specific to the PlayStation. To install (after installing Ghidra), Hit `File > Install Extension > (Plus in the top right of the window) > Ok` then restart the program.
- A Hex Editor [***Recommended: ImHex***](https://imhex.werwolv.net) - We will need to look through the Hex for verification of the files. Once files have been decompressed, text & pointers can be directly visible. ImHex also supports loading the `.tbl` files as well so the SHIFT-JIS can be directly read & copied.
- A [Google Sheet](https://workspace.google.com/products/sheets/) to track the text translations, Pointers, & Offsets(locations)
- [PCSX-Redux](https://github.com/grumpycoders/pcsx-redux) - great emulator with a built-in debugger that will allow us to explore the memory as the game runs. along with support for coding addons.
- [ARMips](https://github.com/Kingcom/armips) - A tool needed for the patching process once translations are ready to applied.

### Decompressing the text

What (I'm assuming) you've done by now:

1. Gotten a (legal) copy of the game & dumped it to your PC
2. Extracted the game's file with [CD Mage](https://www.romhacking.net/utilities/1435/) to a folder.

**Get & Run the BrightisFiler**
Made by Onepiecefreak, the [BrightisFiler](https://github.com/Dickdebonair/Brightis-fan-translation/tree/78d190b5a6b46cbc71955e9e4775a9cb60408ad4/Created%20Tools%20%26%20Other%20Scripts/BrightistFiler) is the best tool currently to decompress two of the files of the game that we are certain contain a majority of the ingame text: `OVR.BIN` & `ONMOVR.BIN`.

You can find out what all our created tools do [here](https://github.com/Dickdebonair/Brightis-fan-translation/tree/78d190b5a6b46cbc71955e9e4775a9cb60408ad4/Created%20Tools%20%26%20Other%20Scripts).  
You can learn how to operate the tool on it's README [here](https://github.com/Dickdebonair/Brightis-fan-translation/tree/78d190b5a6b46cbc71955e9e4775a9cb60408ad4/Created%20Tools%20%26%20Other%20Scripts/BrightistFiler).  

Once you've ran the tool on `OVR.BIN` & `ONMOVR.BIN`, verify that you have 61 `OVR.BIN` parts, and 28 `ONMOVR.BIN` parts. Some parts **WILL** be 0KB. This is intentional.

### Getting the pointers

What (I'm assuming) you've done by now:

- Setup your Hex Editor, Ghidra

#### Installing the PSX Loader

1. Grab the PSX loader Extension [here](https://github.com/lab313ru/ghidra_psx_ldr)
2. File > Install Extensions > Hit the Plus in the top right of the new window
3. Select the .zip of the 'ghidra_psx_ldr' & check the box in the "Install Extensions" window. Restart Ghidra.
4. Create/Open the project using one of the full files from the game [ONMOVR/OVR/SCPS_101.05] in the CodeBrowser.
5. Under 'Tools', "PSX Overlay Manager" will be available.

#### Adding an Overlay

1. Load SCPS_101.05, Overlay each partial OVR/ONMOVR file at the correct starting address (found on the [Google Sheet](https://docs.google.com/spreadsheets/d/16ST1GpUGnfzQkkyA7Y5LqPaeRHxq0L23jmVaQDX_wBU/edit?usp=sharing)) in the header/top row. 
2. Under 'Tools', select "PSX Overlay Manager"
3. Paste in the listed starting overlay address into the "Start Address" field. Rename the "Block Name" to the corresponding partial Overlay file. 
4. Hit "Create from Binary" & select the partial Overlay file the alligns with the name we entered/on the sheet. 

This is where we could use your help!
So as part of our current patching method, we currently need:

1. The **Overlay Pointer** (*"Offset" in our Google sheet*) - where the actual text starts getting loaded into memory. ​
2. The **Data Pointer** (*"Code Data Offset" in our Google sheet*) - where the pointer itself is located when the game is loaded. ​
3. The **Print Pointer** (*"Code Print Offset" in our Google sheet*) - where the function that actually calls the print method is loaded in memory.
4. The **Overlay Entry Point** - where the whole overlay file executes code from.

We have a [YouTube video that covers how to get the 4th here](https://www.youtube.com/watch?v=boKdetWcJ_0), but we're trying to find better ways to get the first 3 programatically.

If you have a method on getting these programatically, or have gathered them manually for the additional parts incomplete on our [Google Sheets](https://docs.google.com/spreadsheets/d/16ST1GpUGnfzQkkyA7Y5LqPaeRHxq0L23jmVaQDX_wBU/edit?usp=sharing) [the later sheets towards the right that are numbered.] Please reach out to me by email or Discord, as listed on the [README Here!](https://github.com/Dickdebonair/Brightis-fan-translation?tab=readme-ov-file#what-were-doing--the-goal)

### Patching in the text

What (I'm assuming) you've done by now:

1. Gotten a google sheet setup for tracking the pointers/text/etc.

**Get & Run the BrightisRenderer**
Made by Onepiecefreak, BrightisRenderer is the custom-made tool to live take in a Google Sheet that has the first 3 of the pointers mentioned above, Text that has been dumped, and render it as close as possible as it would be shown in game.

You can find out what all our created tools do [here](https://github.com/Dickdebonair/Brightis-fan-translation/tree/78d190b5a6b46cbc71955e9e4775a9cb60408ad4/Created%20Tools%20%26%20Other%20Scripts).  
You can learn how to operate the tool on it's README [here](https://github.com/Dickdebonair/Brightis-fan-translation/tree/78d190b5a6b46cbc71955e9e4775a9cb60408ad4/Created%20Tools%20%26%20Other%20Scripts/BrightistRenderer).  

Feel free to use the Google Sheet that we have for the English Patch as a basis for other languages!

**Get & Run TranslationToSource**
Once we have one (or more) of the parts translated, we can use another tool created by Onepiecefreak to convert the sheet into a script, usable by [ARMips](https://github.com/Kingcom/armips), a tool we mentioned at the start.  
You can learn how to operate the tool on it's README [here](https://github.com/Dickdebonair/Brightis-fan-translation/tree/78d190b5a6b46cbc71955e9e4775a9cb60408ad4/Created%20Tools%20%26%20Other%20Scripts/TranslationToSource).

Once completed, the final steps are to run the scripts for each part against [ARMips](https://github.com/Kingcom/armips), and collect them for the next step below!

### Patching the game

What (I'm assuming) you've done by now:

1. Collected all ARMips patched parts for each file.

**Recompressing the files using BrightisFiler**
Just as we did for decompressing the game, BrightisFiler has a 'Re-compress' function. Use that as directed on the the tool's [README](https://github.com/Dickdebonair/Brightis-fan-translation/tree/78d190b5a6b46cbc71955e9e4775a9cb60408ad4/Created%20Tools%20%26%20Other%20Scripts/BrightistFiler), and gather the files.

**Reinserting the modified files to the game**
Once the files have been compressed, we're finally ready to get things back into the game.  
Best way to do this is using CD Mage.

0. BACK UP THE ORGINAL .BIN FILE
1. Open the game the same way we extracted
2. Use the "Insert Files" button, and select the patched files we have.  
Make sure the patched files have the same names and in all Uppercase as the game.
3. Use "Save as" to save the modified BIN, and verify the game runs with the new BIN.

Playtesting!
Do you need an explanation? Get testing that everything WORKS!

### Building a Resuable Patch - Ideas

What (I'm assuming) you've done by now:

1. Gotten a WORKING translation of the game assembled.

We have a number of tools we'd like to investigate.  
Namely, RomHacking.Net's online IPS Patcher & a tool called XDelta.

We haven't locked down which to use, the size differences, or what would be the best to use for the final end user, so investifation of that when we're done will be useful. But only AFTER we've gotten a working translation to play through & completed.
