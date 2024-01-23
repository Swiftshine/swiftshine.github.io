# GFA documentation

Table of contents:
- [GFA documentation](#gfa-documentation)
  - [GFAC - GoodFeelArChive](#gfac---goodfeelarchive)
  - [File Entries](#file-entries)
  - [String Filenames](#string-filenames)
  - [GFCP - GoodFeelComPression](#gfcp---goodfeelcompression)
  - [Data](#data)
  - [Extra](#extra)
    - [What about existing documentation?](#what-about-existing-documentation)
    - [Undocumented GFA Info](#undocumented-gfa-info)

**GoodFeelArchive**:
A file format used in GoodFeel games such as Kirby's Epic Yarn and Yoshi's Wooly World (and their 3DS rereleases).  Byte order is **little endian**.

It is split into five parts: the archive file header, the file entries, the list of strings (for filenames), the compressed data file header, and the compressed data itself.

## GFAC - GoodFeelArChive
Information about the entire archive is defined here.

Header size: **0x30**<sup>1</sup>
<!-- i kinda just gave up with the indention halfway through  -->
| Name      | Offset    | Size      | Data Type     | Description   | 
| ---       | ---       | ---       | ---           | ---           |
| magic     | 0x0       | 0x4       | char[0x4]     | "GFAC"        |
| _4        | 0x4       | 0x4       | u32           | unknown. 0x00030000 in Kirby's Epic Yarn (Wii), 0x01030000 everywhere else.      |
| version   | 0x8       | 0x4       | u32           | always 1.|
| fileCountOffset | 0xC | 0x4 | u32 | offset (relative to 0x0) to the value indicating the total number of files. this value is usually 0x2C. | 
| fileInfoSize | 0x10 | 0x4 | u32 | the size of the file information.<sup>2</sup> |
| dataOffset | 0x14 | 0x4 | u32 | offset (relative to 0x0) to the GFCP header. |
| dataSize | 0x18 | 0x4 | u32 | size of compressed data (GFCP header onwards). |
| padding | 0x1C | 0x10 | u8[0x10] | always 0. | 
| fileCount | 0x2C | 0x4 | u32 | the number of files in the archive. |

1 - The size of the header is actually `0x20`, because neither `padding` nor `fileCount` are part of the archive header (defined in Kirby's Epic Yarn's code); `fileCount` is part of a "fileInfo" header of variable size (which is why I included `fileCount` as part of the archive header).

2 - `fileInfoSize` = size of (`fileCount`) + size of (all file entries) + size of (null-terminated strings).

## File Entries
Information about each file is defined here.

Entry size: **0x10**
| Name      | Offset    | Size      | Data Type     | Description   |
| ---       | ---       | ---       | ---           | ---           |
| hash      | 0x0       | 0x4       | u32           | See below.<sup>3</sup>        |
| nameOffset<sup>4</sup> | 0x4      | 0x4       | u32           | offset (relative to 0x0) to this file's name. |
| size      | 0x8       | 0x4       | u32           | size of decompressed data. |
| dataOffset | 0xC      | 0x4       | u32           | offset (relative to 0x0) to the compressed data.|

3 - ~~This CRC32 hash is calculated with the *uncompressed* data.~~ It's unknown what's actually used to calculate the data, or how it's calculated. [Current tools suggest a CRC32 may be calculated with uncompressed data](https://github.com/jam1garner/gfa-packer/blob/master/gfa-packer.py#L33C5-L33C35).

4 - In the few tools that have anything to do with GFA, `nameOffset` is ANDed with `0x00FFFFFF` when getting the string.

## String Filenames
After the final file entry, the strings are written. These (null-terminated) strings are written consecutively, and once the final string is written, there is padding until the next multiple of `0x10`.


## GFCP - GoodFeelComPression
Information about the compressed data is defined here (meaning of the magic is unknown).

Header size: **0x14**

| Name      | Offset    | Size      | Data Type     | Description   |
| ---       | ---       | ---       | ---           | ---           |
| magic     | 0x0       | 0x4       | char[0x4]     | "GFCP"        |
| version   | 0x4       | 0x4       | u32           | usually 1.    |
| compressionType | 0x8 | 0x4       | u32           | 1 - Byte Pair Encoding. 2 or 3 - LZ77.|
| decompressedSize | 0xC | 0x4 | u32 | size of the (collective) decompressed data.|
| compressedSize | 0x10 | 0x4 | u32 | size of the (collective) compressed data.|

## Data
Data seems to be written directly after `compressedSize`. The data of all files is packed next to each other.

To determine how much data to read for each file:

```cpp
/* (pseudocode) */

GFACHeader gfacHeader;

// you can replace this with your own implementation.
// assume this reader's file is the *uncompressed* data.
FileReader reader; 

int fileCount = gfacHeader.fileCount;
FileEntries fileEntries[fileCount];
Buffer fileBuffer[fileCount];

for (int i = 0; i < fileCount; i++) {
    if (fileEntries[i].dataOffset == 0)
        reader.seek(fileEntries[i].dataOffset + sizeof(GFCPHeader));
    else
        reader.seek(fileEntries[i].dataOffset);
    fileBuffer[i].data = reader.readData(fileEntries[i].size);
}

// each fileBuffer now has its proper data.
```

## Extra
### What about existing documentation?
Documentation on this format has either been found in existing tools, or by myself through several hours of sitting at a hex editor.

Here is a list of everything GFA-related I have found up to this point (in no particular order):

* [This Kirby's Epic Yarn QuickBMS script](http://aluigi.altervista.org/bms/kirby_epic_yarn.bms)
* [This GFA packer for Yoshi's Wooly World](https://github.com/jam1garner/gfa-packer)
* [Switch Toolbox](https://github.com/KillzXGaming/Switch-Toolbox/blob/master/File_Format_Library/FileFormats/Archives/GFA.cs)

### Undocumented GFA Info
* Even when similar games use the same format, there can be individual differences. (Despite these differences, they (*theoretically*) don't matter as long as the offset to the GFCP header matches what is stated in the GFAC header.)
    * In Kirby's Epic Yarn (Wii), there is always padding of `0x10` bytes between the end of the fileInfo (which itself may have padding from the last null-terminated string to the next offset with a multiple of `0x10`) and the GFCP header.
    * In Yoshi's Wooly World (Wii U), the GFCP header is always at `0x2000`.