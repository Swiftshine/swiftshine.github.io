**MNEB**

<sub>(Model NURBS Encoded Binary?)</sub>

The NURBS animation format for *Kirby's Epic Yarn*.

**General Information**
- Byte order: big-endian
- Sections: 3<sup>1</sup>
    - header
    - curve block
    - demo data block

1 - Curve blocks and demo data blocks are typically never seen together.

## Format documentation
### Header
Size: `0x18`

| field              | offset | size  | data type | description                                                                              |
| ------------------ | ------ | ----- | --------- | ---------------------------------------------------------------------------------------- |
| magic              | `0x0`  | `0x4` | `char[4]` | "MNCH" - Model NURBS Control Header?                                                     |
| curve block offset | `0x4`  | `0x4` | `u32`     | Offset to the curve blocks.                                                              |
| unknown            | `0x8`  | `0x4` | `u32`     |                                                                                          |
| curve block count  | `0xC`  | `0x4` | `u32`     | The number of curve blocks.                                                              |
| unknown            | `0x10` | `0x4` | `u32`     |                                                                                          |
| frame count        | `0x14` | `0x2` | `u16`     | The number of frames this animation has.                                                 |
| is animated?       | `0x16` | `0x1` | `bool`    | Indicates if this animation is actually animated -- if false, the curves will be static. |
### Curve Information
#### Curve Block
| field                  | offset | size                        | data type        | description                                                                                                             |
| ---------------------- | ------ | --------------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------- |
| magic                  | `0x0`  | `0x4`                       | `char[4]`        | "MNCN" - Model Nurbs Curve Node?                                                                                        |
| block size             | `0x4`  | `0x4`                       | `u32`            | The size of this block.                                                                                                 |
| name                   | `0x8`  | `0x20`                      | `char[32]`       | The name of this curve.                                                                                                 |
| unknown                | `0x28` | `0x64`                      | unknown          |                                                                                                                         |
| unknown                | `0x8C` | `0x4`                       | `float`          |                                                                                                                         |
| unknown                | `0x90` | `0x4`                       | `u32`            |                                                                                                                         |
| unknown                | `0x94` | `0x4`                       | `u32`            |                                                                                                                         |
| unknown                | `0x98` | `0x4`                       | `u32`            |                                                                                                                         |
| control point offset   | `0x9C` | `0x4`                       | `u32`            | Offset to the control points.                                                                                           |
| knot offset            | `0xA0` | `0x4`                       | `u32`            | Offset to the knots.                                                                                                    |
| key frame info offset  | `0xA4` | `0x4`                       | `u32`            | Offset to the key frame tables.                                                                                         |
| unknown                | `0xA8` | `0x10`                      | `float[4]`       |                                                                                                                         |
| unknown                | `0xB8` | `0x10`                      | `u32[4]`         |                                                                                                                         |
| control point count    | `0xC8` | `0x4`                       | `u32`            | The number of control points                                                                                            |
| control points         | `0xCC` | `8 * [control point count]` | `ControlPoint[]` | The control points.                                                                                                     |
| knot count             |        | `0x4`                       | `u32`            | The number of knots.                                                                                                    |
| knot vector            |        | `4 * [knot count]`          | `float[]`        | The knot vector.                                                                                                        |
| key frame table offset |        | `0x4`                       | `u32`            | The offset to the key frame table.<br><br>The address of this field is what is pointed to by the key frame info offset. |
| explicit padding       |        | `0xC`                       |                  | Ensures that the previous field and the next field have the same alignment.                                             |
| key frame set          |        | variable size               | `KeyFrameSet[]`  | The key frame sets.                                                                                                     |
| key frame set count    |        | `0x4`                       | `u32`            | The number of key frame sets.<br><br>The address of this field is what is pointed to by the key frame table offset.     |
| key frame set offsets  |        | `4 * [key frame set count]` | `u32[]`          | Offsets to the key frame sets.                                                                                          |
#### Control Point

| field | offset | size  | data type | description |
| ----- | ------ | ----- | --------- | ----------- |
| x     | `0x0`  | `0x2` | `s16`     | X position. |
| y     | `0x2`  | `0x2` | `s16`     | Y position. |
| z     | `0x4`  | `0x2` | `s16`     | Z position. |
| w     | `0x6`  | `0x2` | `s16`     | Weight.     |
#### Key Frame
| field      | offset | size  | data type | description                               |
| ---------- | ------ | ----- | --------- | ----------------------------------------- |
| frame      | `0x0`  | `0x2` | `s16`     | The frame this key frame represents.      |
| is active? | `0x2`  | `0x1` | `bool`    | Indicates if an animation is in progress. |
| x          | `0x4`  | `0x2` | `s16`     | X position.                               |
| y          | `0x6`  | `0x2` | `s16`     | Y position.                               |
#### Key Frame Set

| field           | offset | size                    | data type    | description                            |
| --------------- | ------ | ----------------------- | ------------ | -------------------------------------- |
| node index      | `0x0`  | `0x2`                   | `u16`        | The node this key frame set acts upon. |
| key frame count | `0x2`  | `0x2`                   | `u16`        | The number of key frames.              |
| key frames      | `0x4`  | `8 * [key frame count]` | `KeyFrame[]` | The key frames for this key frame set. |
### Demo Information
#### Demo Data Block
| field                   | offset | size                          | data type | description                     |
| ----------------------- | ------ | ----------------------------- | --------- | ------------------------------- |
| magic                   | `0x0`  | `0x4`                         | `char[4]` | "MNDD" - Model NURBS Demo Data? |
| block size              | `0x4`  | `0x4`                         | `u32`     | The size of this block.         |
| demo option set count   | `0x8`  | `0x4`                         | `u32`     | The number of demo option sets. |
| demo option set offsets | `0xC`  | `4 * [demo option set count]` | `u32[]`   | Offsets to demo option sets.    |
#### Demo Option Set
| field               | offset | size                 | data type   | description                  |
| ------------------- | ------ | -------------------- | ----------- | ---------------------------- |
| name                | `0x0`  | `0x20`               | `char[32]`  | The name of this option set. |
| unknown             | `0x20` | `0x20`               | `char[32]`? |                              |
| option count        | `0x40` | `0x4`                | `u32`       | The number of demo options.  |
| demo option offsets | `0x44` | `4 * [option count]` | `u32[]`     | Offsets to demo options.     |
#### Demo Option
| field       | offset | size                | data type  | description                           |
| ----------- | ------ | ------------------- | ---------- | ------------------------------------- |
| name        | `0x0`  | `0x10`              | `char[16]` | The name of this option.              |
| value count | `0x10` | `0x4`               | `u32`      | The number of values for this option. |
| values      | `0x14` | `1 * [value count]` | `u8[]`     | The values for this option.           |

An alignment note: the address of the next `DemoOption` is padded to a multiple of 4 that *isn't* directly after the end of this struct. E.g. if this struct ends at `0x_2`, the next `DemoOption` will start at `0x_4`, but if this struct ends at `0x_4`, the next `DemoOption` will start at `0x_8` instead.
