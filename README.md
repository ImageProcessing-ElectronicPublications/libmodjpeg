# libmodjpeg

A library for JPEG masking and composition in the DCT domain.

## Background

With libmodjpeg you can overlay a (masked) image onto an existing JPEG as lossless as possible. Changes in the JPEG only
take place where the overlayed image is applied. All modifications happen in the DCT domain, thus the JPEG is decoded and
encoded losslessly.

Adding an overlay (e.g. logo, watermark, ...) to an existing JPEG image usually will result in loss of quality because the JPEG
needs to get decoded and then re-encoded after the overlay has been applied. [Read more about JPEG on Wikipedia](https://en.wikipedia.org/wiki/JPEG)

libmodjpeg avoids the decoding and re-encoding of the JPEG image by applying the overlay directly on the un-transformed DCT
coefficients. Only the area where the overlay is applied to is affected by changes and the rest of the image will remain untouched.

The usual process of applying a (masked) overlay involved these steps:

1. Huffman decode
2. de-quantize
3. inverse DCT *
3. colorspace transformation from YCbCr to RGB *
4. applying the (masked) overlay
5. colorspace transformation from RGB to YCbCr *
6. DCT *
7. quantize *
8. Huffman encode

The steps marked with a * will lead to loss of quality.

libmodjpeg avoids all lossy steps by applying the (masked) overlay directly in the DCT domain:

1. Huffman decode
2. de-quantize
3. applying the (masked) overlay
4. quantize
5. Huffman encode

In step 4, the quantization is lossless compared to the usual process because the same DCT and quantization
values are used as in step 2.

Only the overlay itself will experience a loss of quality because it needs to be transformed into the DCT domain
with the same colorspace and sampling as the image it will be applied to.

## Compiling and installing libmodjpeg

libmodjpeg requires the libjpeg v9 6? 7? 8? or compatible (libjpeg-turbo or mozjpeg), however the IJG libjpeg or
libjpeg-turbo are recommended because mozjpeg will always produce progressive JPEGs which may not be desired.

```
# git clone https://github.com/ioppermann/libmodjpeg.git
# cd libmodjpeg
# cmake .
# make
# make install
```

In case libjpeg (or compatible) are installed in a non-standard location you can set the environment variable `CMAKE_PREFIX_PATH`
to the location where libjpeg is installed:

```
# env CMAKE_PREFIX_PATH=/usr/local/opt/jpeg-turbo/ cmake .
```

## Synopsis

```C
#include <libmodjpeg.h>
```

Include the header file in order to have access to the library.

```C
struct mj_dropon_t;
```

A "dropon" denotes the overlay that will be applied to an image and is defined by the struct `mj_dropon_t`.

```C
void mj_init_dropon(mj_dropon_t *d);
```

Initialize the dropon in order to be ready for use.

```C
int  mj_read_dropon_from_buffer(mj_dropon_t *d, const char *rawdata, unsigned int colorspace, size_t width, size_t height, short blend);
```

Read a dropon from a buffer. The buffer is an array of chars holding the raw image data in the given color space

```C
#define MJ_COLORSPACE_RGB               1  // [0] = R0, [1] = G0, [2] = B0, [3] = R1, ...
#define MJ_COLORSPACE_RGBA              2  // [0] = R0, [1] = G0, [2] = B0, [3] = A0, [4] = R1, ...
#define MJ_COLORSPACE_GRAYSCALE         3  // [0] = Y0, [1] = Y1, ...
#define MJ_COLORSPACE_GRAYSCALEA        4  // [0] = Y0, [1] = A0, [2] = Y1, ...
#define MJ_COLORSPACE_YCC               5  // [0] = Y0, [1] = Cb0, [2] = Cr0, [3] = Y1, ...
#define MJ_COLORSPACE_YCCA              6  // [0] = Y0, [1] = Cb0, [2] = Cr0, [3] = A0, [4] = Y1, ...
```

`width` and `height` are dimensions of the raw image. `blend` is a value in [0, 255] for the translucency for the dropon if no alpha
channel is given, where 0 is fully transparent (the dropon will not be applied) and 255 is fully opaque.

```C
int  mj_read_dropon_from_jpeg(mj_dropon_t *d, const char *filename, const char *mask, short blend);
```

Read a dropon from a JPEG file (`filename`). The alpha channel is given by the second JPEG file (`mask`). Use `NULL` is no alpha
channel is available or wanted. `blend` is a value for the translucency for the dropon if no alpha channel is given.

```C
void mj_free_dropon(mj_dropon_t *d);
```

Free the memory consumed by the dropon. The dropon struct can be reused by applying `mj_init_dropon` to it.


```C
struct mj_jpeg_t;

void mj_init_jpeg(mj_jpeg_t *m);
int  mj_read_jpeg_from_buffer(mj_jpeg_t *m, const char *buffer, size_t len);
int  mj_read_jpeg_from_file(mj_jpeg_t *m, const char *filename);

int  mj_write_jpeg_to_buffer(mj_jpeg_t *m, char **buffer, size_t *len, int options);
int  mj_write_jpeg_to_file(mj_jpeg_t *m, char *filename, int options);

void mj_free_jpeg(mj_jpeg_t *m);

int  mj_compose(mj_jpeg_t *m, mj_dropon_t *d, unsigned int align, int offset_x, int offset_y);

int  mj_effect_grayscale(mj_jpeg_t *m);
int  mj_effect_pixelate(mj_jpeg_t *m);
int  mj_effect_tint(mj_jpeg_t *m, int cb_value, int cr_value);
int  mj_effect_luminance(mj_jpeg_t *m, int value);
```

## Using libmodjpeg

```C
struct mj_dropon_t d;

mj_init_dropon(&d);
mj_read_dropon_from_jpeg(&d, "logo.jpg", NULL, 50);
```

```C
struct mj_jpeg_t m;

mj_init_jpeg(&m);
mj_read_jpeg_from_file(&m, "in.jpg");
```

```C
mj_compose(&m, &d, MJ_ALIGN_BOTTOM | MJ_ALIGN_RIGHT, -10, -10);
```

`MJ_ALIGN_LEFT`, `MJ_ALIGN_RIGHT`, `MJ_ALIGN_TOP`, `MJ_ALIGN_BOTTOM`, `MJ_ALIGN_CENTER`

```C
mj_write_jpeg_to_file(&m, "out.jpg", MJ_OPTION_OPTIMIZE | MJ_OPTION_PROGRESSIVE);
```

`MJ_OPTION_NONE`, `MJ_OPTION_OPTIMIZE`, `MJ_OPTION_PROGRESSIVE`, `MJ_OPTION_ARITHMETRIC`

```C
mj_free_jpeg(&m);
mj_free_dropon(&d);
````

## References

[1] R. Jonsson, "Efficient DCT Domain Implementation of Picture Masking
    and Composition and Compositing", ICIP (2) 1997, pp. 366-369

[2] jpeglib http://www.ijg.org/