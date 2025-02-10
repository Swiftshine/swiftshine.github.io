The *Kirby's Epic Yarn* level format is separated into two parts: `mapbin` (for level contents, excluding enemies) and `enbin` (for exclusively enemies).
## Common structures
`Vec2f`:
```cpp
struct Vec2f {
	float x;
	float y;
};
```
`Vec3f`:
```cpp
struct Vec3f {
	float x;
	float y;
	float z;
};
```
`String32`,`String64`:
```cpp
using String32 = char[32];
using String64 = char[64];
```
`CountOffsetPair`:
```cpp
struct CountOffsetPair {
	u32 count;
	u32 offset; // relative to the start of the file
};
```
`MapdataParams`:
```cpp
struct MapdataParams {
	int intParams[3];
	float floatParams[3];
	String64 stringParams[3];
}; // size = 0xD8
```
`CommonGimmickParams`:
```cpp
struct CommonGimmickParams {
	int commonIntParams[2];
	float commonFloatParams[2];
	char commonStringParam[8];
	int intParams[5];
	float floatParams[5];
	String64 stringParams[5];
}; // size = 0x178
```
## Mapbin
**General Information**
- Byte order: big-endian
- Sections: 11
	- header
	- walls
	- labeled walls
	- gimmicks
	- common gimmicks
	- paths
	- zones
	- (race) course info
	- common gimmick name bank
	- collision type name bank
	- wall label name bank

This format does not have padding in-between sections. The file is padded with a value of 0 to a size that is a multiple of `0x20`.

### Format Documentation
#### Header
Size = `0x58`

| field                      | offset | size  | data type         | description                                                                                     |
| -------------------------- | ------ | ----- | ----------------- | ----------------------------------------------------------------------------------------------- |
| unknown                    | `0x0`  | `0x4` | `float`           | This value is often seen to be ~`3.3f`. Values lower than this seem to crash the game.          |
| minimum camera position    | `0x4`  | `0x8` | `Vec2f`           | The camera position will not go lower than this.                                                |
| maximum camera position    | `0xC`  | `0x8` | `Vec2f`           | The camera position will not go higher than this.                                               |
| walls                      | `0x14` | `0x8` | `CountOffsetPair` | Terrain collision.                                                                              |
| labeled walls              | `0x1C` | `0x8` | `CountOffsetPair` | Terrain collision with labels for additional processing.                                        |
| common gimmicks            | `0x24` | `0x8` | `CountOffsetPair` | Level entities that are frequently used.                                                        |
| gimmicks                   | `0x2C` | `0x8` | `CountOffsetPair` | Level entities that are less frequently used.                                                   |
| paths                      | `0x34` | `0x8` | `CountOffsetPair` | Dictates paths for entities to follow or act upon.                                              |
| zones                      | `0x3C` | `0x8` | `CountOffsetPair` | Describes an area in the level in which something may occur consistently or conditionally.      |
| course info                | `0x44` | `0x8` | `CountOffsetPair` | Indicates completion for Mara's race.                                                           |
| common gimmick name offset | `0x4C` | `0x4` | `u32`             | The string bank for common gimmick names starts here, relative to the start of the file.        |
| collision type name offset | `0x50` | `0x4` | `u32`             | The string bank for collision type names starts here, relative to the start of the file.        |
| wall label name offset     | `0x54` | `0x4` | `u32`             | The string bank for the labels of labeled walls starts here, relative to the start of the file. |

#### Walls
Internally called `ColDataSeg`.
Size = `0x20`

| field                | offset | size  | data type | description                                                                |
| -------------------- | ------ | ----- | --------- | -------------------------------------------------------------------------- |
| start position       | `0x0`  | `0x8` | `Vec2f`   |                                                                            |
| end position         | `0x8`  | `0x8` | `Vec2f`   |                                                                            |
| normalized vector    | `0x10` | `0x8` | `Vec2f`   | This is what handles proper collision. See calculation below.<sup>1</sup>  |
| index                | `0x18` | `0x4` | `u32`     | The index of the wall.                                                     |
| collision type index | `0x1C` | `0x4` | `u32`     | Referencing the collision type name bank, the index of the collision type. |

1 - The normalized vector is calculated as such:
```cpp
Vec2f GetNormalizedVector(Vec2f start, Vec2f end) {
	Vec2f direction = Vec2f(end.x - start.x, end.y - start.y);
	float magnitude = sqrt(pow(direction.x, 2.0f) + pow(direction.y, 2.0f));
	Vec2f normalized = Vec2f(direction.x / magnitude, direction.y / magnitude);

	Vec2f result = Vec2f(-normalized.y, normalized.x);
	return result;
}
```

#### Labeled Walls
Internally called `ColDataSegLabel`. This builds off of the existing wall structure, with an additional field.
Size = `0x24`

| field       | offset | size  | data type | description                                                   |
| ----------- | ------ | ----- | --------- | ------------------------------------------------------------- |
| label index | `0x20` | `0x4` | `u32`     | Referencing the wall label name bank, the index of the label. |
#### Common Gimmicks
Size = `0x188`

| field      | offset | size    | data type             | description                                                                                                             |
| ---------- | ------ | ------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| name index | `0x0`  | `0x4`   | `u32`                 | Referencing the common gimmick name bank, the index of the common gimmick name. This name is in ShiftJIS and not ASCII. |
| position   | `0x4`  | `0xC`   | `Vec3f`               |                                                                                                                         |
| parameters | `0x10` | `0x178` | `CommonGimmickParams` |                                                                                                                         |

#### Gimmicks
Size = `0x124`

| field      | offset | size   | data type       | description                                                        |
| ---------- | ------ | ------ | --------------- | ------------------------------------------------------------------ |
| name       | `0x0`  | `0x30` | `char[0x30]`    | ASCII.                                                             |
| unknown    | `0x30` | `0x10` | `char[0x10]`    | This is often set to all 0s. It's not clear what this is used for. |
| position   | `0x40` | `0xC`  | `Vec3f`         |                                                                    |
| parameters | `0x4C` | `0xD8` | `MapdataParams` |                                                                    |

#### Paths
Size > `0x11C` as the structure is variable size

| field            | offset  | size                     | data type                 | description                                                                                |
| ---------------- | ------- | ------------------------ | ------------------------- | ------------------------------------------------------------------------------------------ |
| name             | `0x0`   | `0x20`                   | `char[0x20]`              |                                                                                            |
| type             | `0x20`  | `0x20`                   | `char[0x20]`              | (author's note: I remember that this *is* a type field, but I don't remember any examples) |
| params           | `0x40`  | `0xD8`                   | `MapdataParams`           |                                                                                            |
| number of points | `0x118` | `0x4`                    | `u32`                     |                                                                                            |
| points           | `0x11C` | `0x8` * number of points | `Vec2f[number of points]` |                                                                                            |
#### Zone

Size = `0x128`

| field      | offset  | size   | data type       | description |
| ---------- | ------- | ------ | --------------- | ----------- |
| name       | `0x0`   | `0x20` | `char[0x20]`    |             |
| unknown    | `0x20`  | `0x20` | `char[0x20]`    |             |
| params     | `0x40`  | `0xD8` | `MapdataParams` |             |
| zone start | `0x118` | `0x8`  | `Vec2f`         |             |
| zone end   | `0x120` | `0x8`  | `Vec2f`         |             |
#### Course Info
Size = `0x124`

| field    | offset  | size   | data type       | description |
| -------- | ------- | ------ | --------------- | ----------- |
| name     | `0x0`   | `0x20` | `char[0x20]`    |             |
| unknown  | `0x20`  | `0x20` | `char[0x20]`    |             |
| params   | `0x40`  | `0xD8` | `MapdataParams` |             |
| position | `0x118` | `0xC`  | `Vec3f`         |             |

#### Name banks
The structures are as follows:

| field             | offset | size                       | data type                     | description |
| ----------------- | ------ | -------------------------- | ----------------------------- | ----------- |
| number of entries | `0x0`  | `0x4`                      | `u32`                         |             |
| entries           | `0x4`  | `0x20` * number of entries | `String32[number of entries]` |             |

## Enbin
At the time of writing this, the entirety of the `enbin` format isn't well-known. See [enbin ImHex pattern](https://github.com/Swiftshine/key/blob/main/docs/hexpat/enbin.hexpat) for existing documentation.