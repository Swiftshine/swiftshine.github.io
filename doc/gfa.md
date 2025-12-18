**GoodFeelArchive**
A file format used in Good-Feel games released on Nintendo consoles prior to their Nintendo Switch games.

**General Information**
- Byte order: little-endian
- Sections: 5
	- archive header
	- file entries
	- filename list
	- compression header
	- compressed data

Assume padding of bytes with a value of 0 when there is a gap.

## Format documentation
### Archive Header 
Size = `0x30`

| field             | offset | size  | data type   | description                                                                                                       |
| ----------------- | ------ | ----- | ----------- | ----------------------------------------------------------------------------------------------------------------- |
| magic             | `0x0`  | `0x4` | `char[0x4]` | "GFAC" - **G**ood**F**eel **A**r**C**hive                                                                         |
| version           | `0x4`  | `0x4` | `u32`       | Rightmost byte is version major. The byte after that is version minor. The other two bytes don't seem to be used. |
| compressed        | `0x8`  | `0x1` | `bool`      | Indicates whether or not the file has a compression header.                                                       |
| entry info offset | `0xC`  | `0x4` | `u32`       | Offset to entry information.                                                                                      |
| file info size    | `0x10` | `0x4` | `u32`       | The size of the file information.<sup>1</sup>                                                                     |
| gfcp offset       | `0x14` | `0x4` | `u32`       | Offset to GFCP header.                                                                                            |
| payload size      | `0x18` | `0x4` | `u32`       | Size of the rest of the file from the start of the GFCP header onwards.                                           |
| file count        | `0x2C` | `0x4` | `u32`       | The number of files in the archive.                                                                               |
### File Entries
Size = `0x10`

| field               | offset | size  | data type | description                                                                                                                                                     |
| ------------------- | ------ | ----- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| checksum            | `0x0`  | `0x4` | `u32`     | This is based on the filename, not the contents of the file.<sup>2</sup>                                                                                        |
| name offset         | `0x4`  | `0x4` | `u32`     | Offset to the filename. To process, this value is to be `AND`ed with `0x00FFFFFF`. If this is the last entry in the archive, a flag of `0x80000000` is applied. |
| size                | `0x8`  | `0x4` | `u32`     | Size of the file when uncompressed.                                                                                                                             |
| decompressed offset | `0xC`  | `0x4` | `u32`     | See below.<sup>3</sup>                                                                                                                                          |
### Filename Strings
Immediately after the final file entry, null-terminated strings are written consecutively. Once the final string is written, there is padding until the next multiple of `0x10`.

### Compression Header
Size = `0x14`

Before compression, files are concatenated.

| field             | offset | size  | data size   | description                                                       |
| ----------------- | ------ | ----- | ----------- | ----------------------------------------------------------------- |
| magic             | `0x0`  | `0x4` | `char[0x4]` | "GFCP" - **G**ood**F**eel **C**om**P**ression                     |
| version?          | `0x4`  | `0x4` | `u32`       | usually 1.                                                        |
| compression type  | `0x8`  | `0x4` | `u32`       | See chart below.<sup>4</sup>                                      |
| decompressed size | `0xC`  | `0x4` | `u32`       | Size of collective decompressed data, padded to offset of `0x10`. |
| compressed size   | `0x10` | `0x4` | `u32`       | Size of compressed data.                                          |

1 - Calculated this way:
```c++
uint32_t size =
	4 + 
	(entry_count * sizeof(entry)) + 
	(length_of_all_strings); 
	// ^ this includes the null terminator for *every* string
```

2 - Calculated this way:
```c++
const char* filename;
uint32_t checksum = 0;

for (auto i = 0; filename[i] != 0; i++) {
	char c = filename[i];
	checksum = c + checksum * 137;
}
```

3 - This is calculated as if the file structure was this:
- archive header
- file entries
- filename
- decompressed data (padded to a size of `0x10`/`0x20` if necessary)

4 - Compression types:

| value | type                  |
| ----- | --------------------- |
| 1     | Byte Pair Encoding    |
| 2     | Speculated to be LZ10 |
| 3     | LZ10                  |

## Tools
- [Tangle](https://github.com/Swiftshine/Tangle)
- [Rust Crate](https://github.com/Swiftshine/gfarch-rs)
## Existing Documentation
Aside from reverse engineering certain aspects of the format myself, I referenced the following:
- [Kirby's Epic Yarn QuickBMS script](http://aluigi.altervista.org/bms/kirby_epic_yarn.bms)
- [Yoshi's Wooly World GFA Packer](https://github.com/jam1garner/gfa-packer)

