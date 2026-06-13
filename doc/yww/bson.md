**GoodFeel BSON**
A binary JSON used in *Yoshi's Woolly World* and *Poochy and Yoshi's Woolly World*.

**General Information**
- Byte order: platform-dependent (big endian on Wii U, little endian on 3DS)
- Sections: 3
	- Root node
	- String table
	- String bank
- All sections are 4-byte aligned.

## Format documentation

### BSON Header

| field            | offset | size  | data type   | description                                                    |
| ---------------- | ------ | ----- | ----------- | -------------------------------------------------------------- |
| magic            | `0x0`  | `0x4` | `char[0x4]` | "BSON"                                                         |
| version          | `0x4`  | `0x4` | `u32`       | This value is always 3.                                        |
| root node offset | `0x8`  | `0x4` | `u32`       | The offset to the root node, typically right after the header. |
| filesize         | `0xC`  | `0x4` | `u32`       | The size of the entire BSON file.                              |
### Nodes

#### Node Types
```c++
enum NodeType {
	Root 		= 300,
	Object 		= 301,
	Array 		= 302,
	Integer		= 303,
	Float		= 304,
	String		= 305,
	Bool		= 306, // An alias for Integer; 32-bits in size
	StringTable	= 400,
	StringBank	= 500, // UTF-8
	EndOfFile	= 900,
};
```

#### Node Header
Universal to all node types is a node header.

| field     | offset | size  | data type  | description                                                                                              |
| --------- | ------ | ----- | ---------- | -------------------------------------------------------------------------------------------------------- |
| node type | `0x0`  | `0x4` | `NodeType` | The type of the node.                                                                                    |
| node size | `0x4`  | `0x4` | `u32`      | The size of the node, *excluding* this header. If the node type is end of file, this value will be zero. |

All nodes described in [[#Node Layouts]] will also contain a index (`u32` at offset `0x8`) of its key in the string table. Arrays typically get empty keys.
#### Node Layouts
##### Object, Array
Variable size.

| field            | offset | size          | data type           | description                                         |
| ---------------- | ------ | ------------- | ------------------- | --------------------------------------------------- |
| child node count | `0xC`  | `0x4`         | `u32`               | The number of nodes associated with this container. |
| child data       | `0x10` | variable size | variable data types | The nodes associated with this container.           |
##### Integer, Bool
A **signed** 32-bit integer.
##### Float
A 32-bit floating-point value.
##### String
An index (`u32`) in the string table.
##### String Table
An array of `node size / 8` string entries.

```c++
struct StringEntry {
	u32 offset; // relative to the start of the data after the string bank header
	u32 length; // the length of the string
};
```

##### String Bank
Raw string data. The strings are not aligned.

## Tools
- [Fleece](https://github.com/Swiftshine/fleece)
- [Rust Crate](https://github.com/Swiftshine/gfbson)
