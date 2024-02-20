# GFA documentation
This documentation is **NOT COMPLETE** by any means, so be careful if you want to use this for anything. I'll be updating this as I find out more.


Table of contents:
- [GFA documentation](#gfa-documentation)
  - [GFAC - GoodFeelArChive](#gfac---goodfeelarchive)
  - [File Entries](#file-entries)
  - [String Filenames](#string-filenames)
  - [GFCP - GoodFeelComPression](#gfcp---goodfeelcompression)
  - [Extra](#extra)
    - [What about existing documentation?](#what-about-existing-documentation)
    - [Undocumented GFA Info](#undocumented-gfa-info)
    - [GFA packing/unpacking overview](#gfa-packingunpacking-overview)
  - [Data (this section is outdated)](#data-this-section-is-outdated)

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

3 - The hash is calculated with just the file's name, shown below:
```c++
std::string filename;
unsigned int hash = 0;
for (int i = 0; i < filename.length(); i++) {
    char c = filename[i];
    hash = c + hash * 0x89;
}
// you now have a hash calculation
```

4 - In the few tools that have anything to do with GFA, `nameOffset` is ANDed with `0x00FFFFFF` when getting the string.

## String Filenames
After the final file entry, the strings are written. These (null-terminated) strings are written consecutively, and once the final string is written, there is padding until the next multiple of `0x10`.


## GFCP - GoodFeelComPression
Information about the compressed data is defined here (meaning of the magic is unknown).

Header size: **0x14** (might actually just be **0x20**)

| Name      | Offset    | Size      | Data Type     | Description   |
| ---       | ---       | ---       | ---           | ---           |
| magic     | 0x0       | 0x4       | char[0x4]     | "GFCP"        |
| version   | 0x4       | 0x4       | u32           | usually 1.    |
| compressionType | 0x8 | 0x4       | u32           | 1 - Byte Pair Encoding. 2 or 3 - LZ77.|
| decompressedSize | 0xC | 0x4 | u32 | size of the (collective) decompressed data.|
| compressedSize | 0x10 | 0x4 | u32 | size of the (collective) compressed data.|



## Extra
### What about existing documentation?
Documentation on this format has either been found in existing tools, or by myself through several hours of sitting at a hex editor.

Here is a list of everything GFA-related I have found up to this point (in no particular order):

* [This Kirby's Epic Yarn QuickBMS script](http://aluigi.altervista.org/bms/kirby_epic_yarn.bms)
* [This GFA packer for Yoshi's Wooly World](https://github.com/jam1garner/gfa-packer)
* [Switch Toolbox](https://github.com/KillzXGaming/Switch-Toolbox/blob/master/File_Format_Library/FileFormats/Archives/GFA.cs)
* [My own tool](https://github.com/Swiftshine/Tangle)
* * It's a bit flawed, but it works!
### Undocumented GFA Info
* Even when similar games use the same format, there can be individual differences. (Despite these differences, they (*theoretically*) don't matter as long as the offset to the GFCP header matches what is stated in the GFAC header.)
    * In Kirby's Epic Yarn (Wii), there is always padding of `0x10` bytes between the end of the fileInfo (which itself may have padding from the last null-terminated string to the next offset with a multiple of `0x10`) and the GFCP header.
    * In Yoshi's Wooly World (Wii U), the GFCP header is always at `0x2000`.


### GFA packing/unpacking overview
While you wait for me to RE the games to figure out more about the format, I'll write some <ins>pseudocode</ins> to demonstrate what I currently believe to be the process:

**Packing**

```cpp
int fileCount;

// assume that a 'FileBuffer' is just raw data for a file
FileBuffer file[fileCount];
FileBuffer result;
FileBuffer all;

// I want to emphasise that the data is aggregated
// first. only then does it get compressed.
for (int i = 0; i < fileCount; i++) {
    all.append(file[i]);
}
FileBuffer compressed = compress(all, compressionType);

GFACHeader gfac;
// assume that here you set up the GFAC header.
// for example:
gfac.fileCount = fileCount;

result.append(gfac);

for (int i = 0; i < fileCount; i++) {
    FileEntry entry;
    entry.hash = calculateHash(appropriate_file_name);
    entry.size = file[i].size;
    // assume you set up the rest of the entry data
    result.append(entry);
}

// atm it's unknown what the conditions are
// for padding to just appear in a file
// but for the most part, it can be ignored
// as long as you set the rest of the fields
// in the GFAC header as such to reflect that
if (padding_is_needed) {
    result.append(appropriate_padding_amount);
}

GFCPHeader gfcp;
// assume that here you set up the GFCP header.
// for example:
gfcp.decompressedSize = all.size;
gfcp.compressedSize   = compressed.size;

result.append(gfcp);
result.append(compressed);

writeFile(result, "cool_file.gfa");
// your new GFA file should (theoretically)
// be complete
```

**Unpacking**

```cpp
FileBuffer archive;
// assume this 'FileReader' does what the name implies
FileReader reader;
reader.setFile(archive);

// assume that any function this FileReader calls
// will use (and update) the reader's current
// seek position in the file data

GFACHeader gfac = reader.read(sizeof(GFACHeader));
int fileCount = gfac.fileCount;

// these will be relevant later
FileEntry fileEntries[];
char* filenames[];

for (int i = 0; i < fileCount; i++) {
    FileEntry entry;
    entry = reader.read(sizeof(FileEntry));

    unsigned int nameOffs;
    // 'nameOffs' is RELATIVE TO THE BEGINNING
    // OF FILE!
    nameOffs = entry.filenameOffset;
    int seekerpos = reader.pos;
    reader.pos = nameOffs & 0x00FFFFFF;
    filenames.append(reader.getString());
    reader.pos = seekerpos;
    fileEntries.append(entry);
}

if (padding_is_present) {
    reader.pos += appropriate_padding_amount;
}

GFCPHeader gfcp = reader.read(sizeof(GFCPHeader));
int compressionType = gfcp.compressionType;

FileBuffer all = reader.readTillEnd();
FileBuffer decompressed;
decompressed = decompress(all, compressionType);

reader.setFile(decompressed);
for (int i = 0; i < fileCount; i++) {
    FileBuffer temp = reader.read(fileEntries[i].size);
    writeFile(temp, filenames[i]);
}

// your extracted files should (theoretically)
// be successfully extracted
```

Obviously there's a lot more to it, some of which I don't know about, but there's a simplification if you ever needed one.

## Data (this section is outdated)
- editor's note, I think this section is wrong now, but I'll keep it here if you want to look at it.

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