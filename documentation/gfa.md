# GFA documentation

**GoodFeelArchive**:
A file format used in GoodFeel games such as Kirby's Epic Yarn and Yoshi's Wooly World (and their 3DS rereleases).  Byte order is **little endian**.

It is split into five parts: the archive file header, the file entries, the list of strings (for filenames), the compressed data file header, and the compressed data itself.

## GFAC - GoodFeelArChive
Header size: **0x30**
<!-- i kinda just gave up with the indention halfway through  -->
| Name      | Offset    | Size      | Data Type     | Description   | 
| ---       | ---       | ---       | ---           | ---           |
| magic     | 0x0       | 0x4       | char[0x4]     | "GFAC"        |
| _4        | 0x4       | 0x4       | u32           | unknown.      |
| version   | 0x4       | 0x4       | u32           | this value is different between games for some reason.|
| fileInfoOffset | 0xC | 0x4 | u32 | offset (relative to 0x0) to the value indicating the total number of files. this value is usually 0x2C. | 
| fileInfoSize | 0x10 | 0x4 | u32 | (insert description) |
| dataOffset | 0x14 | 0x4 | u32 | offset (relative to 0x0) to the GFCP header. |
| dataSize | 0x18 | 0x4 | u32 | (insert description) |
| padding | 0x1C | 0x10 | u8[0x10] | always 0. | 
| fileCount | 0x2C | 0x4 | u32 | the number of files in the archive. |

## File Entries
Entry size: **0x10**
| Name      | Offset    | Size      | Data Type     | Description   |
| ---       | ---       | ---       | ---           | ---           |
| hash      | 0x0       | 0x4       | u32           | CRC32.        |
| nameOffset\* | 0x4      | 0x4       | u32           | offset (relative to 0x0) to this file's name |
| size      | 0x8       | 0x4       | u32           | size of compressed data |
| dataOffset | 0xC      | 0x4       | u32           | offset (relative to 0x0) to the compressed data|

\* In the few tools that have anything to do with GFA, `nameOffset` is ANDed with `0x00FFFFFF` when getting the string.

## String filenames
After the final file entry, the strings are written. These (null-terminated) strings are written consecutively, and once the final string is written, there is padding until the next multiple of `0x10`.


## GFCP - GoodFeelComPression
(actual meaning unknown)

Header size: **0x14**

| Name      | Offset    | Size      | Data Type     | Description   |
| ---       | ---       | ---       | ---           | ---           |
| magic     | 0x0       | 0x4       | char[0x4]     | "GFCP"        |
| version   | 0x4       | 0x4       | u32           | usually 1.    |
| compressionType | 0x8 | 0x4       | u32           | 1 - Byte Pair Encoding. 2 or 3 - LZ77.|
| decompressedSize | 0xC | 0x4 | u32 | size of the (collective!) decompressed data.|
| compressedSize | 0x10 | 0x4 | u32 | size of the (collective!) compressed data.|

## Data
Data seems to be written directly after `compressedSize`. The data of all files is packed next to each other.

To determine how much data to read for each file:

```cpp
/* (pseudocode) */

GFACHeader gfacHeader;
FileReader reader; // you can replace this with your own implementation.

int fileCount = gfacHeader.fileCount;
FileEntries fileEntries[fileCount];
Buffer fileBuffer[fileCount];

for (int i = 0; i < fileCount; i++) {
    // get the offset of the compressed data relative to 0x0
    unsigned int dataPos = fileEntries[i].dataOffset - gfacHeader.dataOffset;

    reader.seek(dataPos);

    fileBuffer[i].data = reader.readData(fileEntries[i].size);
}

// each fileBuffer now has its proper data.
```