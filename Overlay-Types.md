---
title: Overlay Types
layout: default
parent: General Info & Common Notes
nav order: 2
---

## General Overlay Reading

The Playstation 1 has a very generalized recommendation on creating/drawing text and other non-3D visual info onto the screen that most developers, including Quintet, have followed.

These general guidelines & technical documentation can be found here: <https://www.beneaththewaves.net/Software/TDR_Practice_Using_OVERLAYS.html#WalkthroughOVERLAYSOverlays>

A working code example, similar to one used in Brightis, can be found here: <https://github.com/JaberwockySeamonstah/PSXOverlayExample>

## Base (Dialogue) Overlay

90 percent of the text in the game is found in the "Dialogue" or Base Overlay.  
The `BaseOverlay` is found at `80158138` in the System memory.  
Some larger conversations will utilize the Progressive Overlay (noted as 'ProgOverlay' in technical docs) found at `80153EB0`.
We have these denoted as "Text Type 0" in the Spreadsheet.

The Game draws a `RECT` with a blue background, references images for the borders along the sides and corners, and calls a `PrintText` function to convert the raw script into characters that will be printed onscreen.  
In the middle of the script, there are various "Control Codes" that set flags, line breaks, prompt the user for input, clears the text box, or trigger other events.

<img width="50%" height="50%" alt="Base Overlay" src="https://github.com/user-attachments/assets/f05aa486-93aa-43a2-88ed-a839817f5234" />

## Menus, Shops, and others

Similar to the Base Overlay, but has much more freeform display methods.  
Typically these overlays will fill the screen as the focus is on the information provided.  
The bottom text box will update based on what's selected.

Shops & Technique screens use the Constant Overlay (noted as `CnstOverlay` in technical docs) found at `8015F328`  

<img src="https://github.com/user-attachments/assets/7e38e050-de22-412a-8a34-58bb83c6eb6d" width=50% height=50%>
<img src="https://github.com/user-attachments/assets/ac48fc00-753f-4148-af77-3bf743b41305" width=50% height=50%>

***

Status screens, such as the pause menu, use the `SubOverlay` found at `80156CA4`.

<img src="https://github.com/user-attachments/assets/7202fd34-4018-424f-96c5-54414a44709b" width=50% height=50%>

## Tutorial and On-Screen Overlays

When the player is instructed to press certain buttons, such as during the tutorial dungeon or the combat training from Regulus, Text is displayed on screen without the `RECT` that's used in all other Overlays.

The Game will still use the Base Overlay adress to show this, but calls a special method in game to display two lines staggered, on the right half of the screen.
We have these denoted as "Text Type 1" in the Spreadsheet.

<img src="https://github.com/user-attachments/assets/e3512539-c1a3-4bd3-93fa-09a3f0f9710d" width=50% height=50%>

## Area Transitions

Finally, when entering a dungeon or special area, the game will show the name of that area on a black baground as it loads.

It appears to use the Base Overlay, when comparing the offset in the file compared to it's location in memory, but we need to work more on it specifics & details, as it appears to use it's own function to space the characters out for dramatic effect.  

We have these denoted as "Text Type 2" in the Spreadsheet (subject to change).

<img src="https://github.com/user-attachments/assets/aba98cba-75bc-4c7c-9cbb-80ef334b2f56" width=50% height=50%>
