---
title: MAP.BIN
layout: default
parent: File Specifics
nav_order: 5
---
# MAP.BIN Specifics

- While it does not have a Table of Contents, its blobs/sectors are just compressed and aligned to `0x800`
- They seem to be a form of data tree
- They contain textures for the world
  - Textures are paletted images.
  - Start with an identifier TEX8 followed by a palette of 256 colors, encoded with RGB555.
  - identifers follows the image data itself, encoded as 1-byte indexes into the palette
  - texture size to be 256x256
- BIN seems to contain more than just one file format
