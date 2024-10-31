**BGST**

The background format for *Kirby's Epic Yarn*.

**General Information**
- Byte order: big-endian
- Sections: 3
	- header
	- grid entries
	- image data

## Format documentation
### Header
Size = `0x40`

| field             | offset | size  | data type   | description                                                                     |
| ----------------- | ------ | ----- | ----------- | ------------------------------------------------------------------------------- |
| magic             | `0x0`  | `0x4` | `char[0x4]` | "BGST"                                                                          |
| unknown           | `0x4`  | `0x4` | `u32`?      | Often seen to be `0x10` or `0x11`.                                              |
| image width       | `0x8`  | `0x4` | `u32`       | The width of each image, in pixels.                                             |
| image height      | `0xC`  | `0x4` | `u32`       | The height of each image, in pixels.                                            |
| grid width        | `0x10` | `0x4` | `u32`       | The number of rows in the grid.                                                 |
| grid height       | `0x14` | `0x4` | `u32`       | The number of columns in the grid.                                              |
| image count       | `0x18` | `0x4` | `u32`       | The number of total images in the file.                                         |
| layer enabled     | `0x1C` | `0xC` | `bool[0xC]` | Indicates which rendering layers are available for the file to use.<sup>1</sup> |
| grid entry offset | `0x28` | `0x4` | `u32`       | Offset to grid entries.                                                         |
| image data offset | `0x2C` | `0x4` | `u32`       | Offset to image data array.                                                     |
| y offset          | `0x30` | `0x4` | `float`     | Indicates a vertical offset when rendering the file contents.                   |

1 - Below is an enum of rendering layers as they are called by the game. Relative to the viewer, they're sorted from furthest to nearest:

```c++
enum Layer {
	Far05  = 0,
	Far04  = 1,
	Far03  = 2,
	Far02  = 3,
	Far01  = 4,
	Map    = 5,
	Game   = 6,
	Near01 = 7,
	Near02 = 8,
	Near03 = 9,
	Near04 = 10,
	Near05 = 11
};
```

### Grid Entries
Grid entries can be thought of as information for rendering a grid cell. Every field here is treated by the game as a signed 16-bit integer.

| field            | offset | description                                                       |
| ---------------- | ------ | ----------------------------------------------------------------- |
| entry enabled    | `0x0`  | Indicates whether or not this entry should be rendered.           |
| layer            | `0x2`  | The layer this entry will be rendered on.                         |
| row              | `0x4`  | The row this entry will be rendered on.                           |
| column           | `0x6`  | The column this entry will be rendered on.                        |
| main image index | `0x8`  | The image this entry will render.<sup>2</sup>                     |
| mask image index | `0xA`  | The mask this entry will apply to this image, if any.<sup>2</sup> |
| unknown          | `0xC`  | This field is almost always `0xFFFF`.                             |
| unknown          | `0xE`  | This field is almost always `0xFFFF`.                             |

2 - Image indices can be assigned a certain value.
- A value greater than or equal to `0` is an image index.
- A value equal to `-1` indicates that no image is to be rendered.
- The meaning of a value equal to `-2` is currently unknown.
Additionally, fields that *appear* to be grid entries do not follow the regular grid entry pattern -- an example can be found in `stage/stage000/section001.bgst3`. It's unknown if this is an entry, some other data, or garbage.

### Image data
A "main image" is a [CMPR](https://wiki.tockdom.com/wiki/Image_Formats#CMPR)-compressed image. Because transparency is not supported in that format, grids that want to render a specific part of the image must use an additional [I4](https://wiki.tockdom.com/wiki/Image_Formats#I4)-encoded alpha mask.

These compressed data blocks are `0x20000` bytes in size.

#### Example of main images and masks
The following examples are from `stage/stage000/section005.bgst3`.


Main image: index 26
![](res/bgst_main_example_1.png) 

Mask image: index 27
![](res/bgst_mask_example_1.png)





Main image: index 28
![](res/bgst_main_example_2.png) 

Mask image: index 29
![](res/bgst_mask_example_2.png)




## Tools
[bgsttool](https://github.com/Swiftshine/bgst)
