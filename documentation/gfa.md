# GFA - GoodFeelArchive
A file format used in GoodFeel games such as Kirby's Epic Yarn and Yoshi's Wooly World (and their 3DS rereleases). byte order is **little endian**.

it is split into five parts: the archive file header, the file entries, the list of strings (for filenames), the compressed data file header, and the compressed data itself.

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
| padding | 0x1C | 0x10 | u8[0x10] | always zeroes. | 
| fileCount | 0x2C | 0x4 | u32 | the number of files in the archive. |

## File Entries

## String filenames

## GFCP - GoodFeelComPression
(actual meaning unknown)

Header size: **???**

| Name      | Offset    | Size      | Data Type     | Description   |
| ---       | ---       | ---       | ---           | ---           |
| magic     | 0x0       | 0x4       | char[0x4]     | "GFCP"        |
## Data
