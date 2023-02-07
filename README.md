Overview
The Python Imaging Library adds image processing capabilities to your Python interpreter.

This library provides extensive file format support, an efficient internal representation, and fairly powerful image processing capabilities.

The core image library is designed for fast access to data stored in a few basic pixel formats. It should provide a solid foundation for a general image processing tool.

Let’s look at a few possible uses of this library.

Image Archives
The Python Imaging Library is ideal for image archival and batch processing applications. You can use the library to create thumbnails, convert between file formats, print images, etc.

The current version identifies and reads a large number of formats. Write support is intentionally restricted to the most commonly used interchange and presentation formats.

Image Display
The current release includes Tk PhotoImage and BitmapImage interfaces, as well as a Windows DIB interface that can be used with PythonWin and other Windows-based toolkits. Many other GUI toolkits come with some kind of PIL support.

For debugging, there’s also a show() method which saves an image to disk, and calls an external display utility.

Image Processing
The library contains basic image processing functionality, including point operations, filtering with a set of built-in convolution kernels, and colour space conversions.

The library also supports image resizing, rotation and arbitrary affine transforms.

There’s a histogram method allowing you to pull some statistics out of an image. This can be used for



Tutorial
Using the Image class
The most important class in the Python Imaging Library is the Image class, defined in the module with the same name. You can create instances of this class in several ways; either by loading images from files, processing other images, or creating images from scratch.

To load an image from a file, use the open() function in the Image module:

from PIL import Image
im = Image.open("hopper.ppm")
If successful, this function returns an Image object. You can now use instance attributes to examine the file contents:

print(im.format, im.size, im.mode)
PPM (512, 512) RGB
The format attribute identifies the source of an image. If the image was not read from a file, it is set to None. The size attribute is a 2-tuple containing width and height (in pixels). The mode attribute defines the number and names of the bands in the image, and also the pixel type and depth. Common modes are “L” (luminance) for greyscale images, “RGB” for true color images, and “CMYK” for pre-press images.

If the file cannot be opened, an OSError exception is raised.

Once you have an instance of the Image class, you can use the methods defined by this class to process and manipulate the image. For example, let’s display the image we just loaded:

im.show()
Note

The standard version of show() is not very efficient, since it saves the image to a temporary file and calls a utility to display the image. If you don’t have an appropriate utility installed, it won’t even work. When it does work though, it is very handy for debugging and tests.

The following sections provide an overview of the different functions provided in this library.

Reading and writing images
The Python Imaging Library supports a wide variety of image file formats. To read files from disk, use the open() function in the Image module. You don’t have to know the file format to open a file. The library automatically determines the format based on the contents of the file.

To save a file, use the save() method of the Image class. When saving files, the name becomes important. Unless you specify the format, the library uses the filename extension to discover which file storage format to use.

Convert files to JPEG
import os, sys
from PIL import Image

for infile in sys.argv[1:]:
    f, e = os.path.splitext(infile)
    outfile = f + ".jpg"
    if infile != outfile:
        try:
            with Image.open(infile) as im:
                im.save(outfile)
        except OSError:
            print("cannot convert", infile)
A second argument can be supplied to the save() method which explicitly specifies a file format. If you use a non-standard extension, you must always specify the format this way:

Create JPEG thumbnails
import os, sys
from PIL import Image

size = (128, 128)

for infile in sys.argv[1:]:
    outfile = os.path.splitext(infile)[0] + ".thumbnail"
    if infile != outfile:
        try:
            with Image.open(infile) as im:
                im.thumbnail(size)
                im.save(outfile, "JPEG")
        except OSError:
            print("cannot create thumbnail for", infile)
It is important to note that the library doesn’t decode or load the raster data unless it really has to. When you open a file, the file header is read to determine the file format and extract things like mode, size, and other properties required to decode the file, but the rest of the file is not processed until later.

This means that opening an image file is a fast operation, which is independent of the file size and compression type. Here’s a simple script to quickly identify a set of image files:

Identify Image Files
import sys
from PIL import Image

for infile in sys.argv[1:]:
    try:
        with Image.open(infile) as im:
            print(infile, im.format, f"{im.size}x{im.mode}")
    except OSError:
        pass
Cutting, pasting, and merging images
The Image class contains methods allowing you to manipulate regions within an image. To extract a sub-rectangle from an image, use the crop() method.

Copying a subrectangle from an image
box = (100, 100, 400, 400)
region = im.crop(box)
The region is defined by a 4-tuple, where coordinates are (left, upper, right, lower). The Python Imaging Library uses a coordinate system with (0, 0) in the upper left corner. Also note that coordinates refer to positions between the pixels, so the region in the above example is exactly 300x300 pixels.

The region could now be processed in a certain manner and pasted back.

Processing a subrectangle, and pasting it back
region = region.transpose(Image.Transpose.ROTATE_180)
im.paste(region, box)
When pasting regions back, the size of the region must match the given region exactly. In addition, the region cannot extend outside the image. However, the modes of the original image and the region do not need to match. If they don’t, the region is automatically converted before being pasted (see the section on Color transforms below for details).

Here’s an additional example:

Rolling an image
def roll(im, delta):
    """Roll an image sideways."""
    xsize, ysize = im.size

    delta = delta % xsize
    if delta == 0:
        return im

    part1 = im.crop((0, 0, delta, ysize))
    part2 = im.crop((delta, 0, xsize, ysize))
    im.paste(part1, (xsize - delta, 0, xsize, ysize))
    im.paste(part2, (0, 0, xsize - delta, ysize))

    return im
Or if you would like to merge two images into a wider image:

Merging images
def merge(im1, im2):
    w = im1.size[0] + im2.size[0]
    h = max(im1.size[1], im2.size[1])
    im = Image.new("RGBA", (w, h))

    im.paste(im1)
    im.paste(im2, (im1.size[0], 0))

    return im
For more advanced tricks, the paste method can also take a transparency mask as an optional argument. In this mask, the value 255 indicates that the pasted image is opaque in that position (that is, the pasted image should be used as is). The value 0 means that the pasted image is completely transparent. Values in-between indicate different levels of transparency. For example, pasting an RGBA image and also using it as the mask would paste the opaque portion of the image but not its transparent background.

The Python Imaging Library also allows you to work with the individual bands of an multi-band image, such as an RGB image. The split method creates a set of new images, each containing one band from the original multi-band image. The merge function takes a mode and a tuple of images, and combines them into a new image. The following sample swaps the three bands of an RGB image:

Splitting and merging bands
r, g, b = im.split()
im = Image.merge("RGB", (b, g, r))
Note that for a single-band image, split() returns the image itself. To work with individual color bands, you may want to convert the image to “RGB” first.

Geometrical transforms
The PIL.Image.Image class contains methods to resize() and rotate() an image. The former takes a tuple giving the new size, the latter the angle in degrees counter-clockwise.

Simple geometry transforms
out = im.resize((128, 128))
out = im.rotate(45) # degrees counter-clockwise
To rotate the image in 90 degree steps, you can either use the rotate() method or the transpose() method. The latter can also be used to flip an image around its horizontal or vertical axis.

Transposing an image
out = im.transpose(Image.Transpose.FLIP_LEFT_RIGHT)
out = im.transpose(Image.Transpose.FLIP_TOP_BOTTOM)
out = im.transpose(Image.Transpose.ROTATE_90)
out = im.transpose(Image.Transpose.ROTATE_180)
out = im.transpose(Image.Transpose.ROTATE_270)
transpose(ROTATE) operations can also be performed identically with rotate() operations, provided the expand flag is true, to provide for the same changes to the image’s size.

A more general form of image transformations can be carried out via the transform() method.

Color transforms
The Python Imaging Library allows you to convert images between different pixel representations using the convert() method.

Converting between modes
from PIL import Image

with Image.open("hopper.ppm") as im:
    im = im.convert("L")
The library supports transformations between each supported mode and the “L” and “RGB” modes. To convert between other modes, you may have to use an intermediate image (typically an “RGB” image).

Image enhancement
The Python Imaging Library provides a number of methods and modules that can be used to enhance images.

Filters
The ImageFilter module contains a number of pre-defined enhancement filters that can be used with the filter() method.

Applying filters
from PIL import ImageFilter
out = im.filter(ImageFilter.DETAIL)
Point Operations
The point() method can be used to translate the pixel values of an image (e.g. image contrast manipulation). In most cases, a function object expecting one argument can be passed to this method. Each pixel is processed according to that function:

Applying point transforms
# multiply each pixel by 1.2
out = im.point(lambda i: i * 1.2)
Using the above technique, you can quickly apply any simple expression to an image. You can also combine the point() and paste() methods to selectively modify an image:

Processing individual bands
# split the image into individual bands
source = im.split()

R, G, B = 0, 1, 2

# select regions where red is less than 100
mask = source[R].point(lambda i: i < 100 and 255)

# process the green band
out = source[G].point(lambda i: i * 0.7)

# paste the processed band back, but only where red was < 100
source[G].paste(out, None, mask)

# build a new multiband image
im = Image.merge(im.mode, source)
Note the syntax used to create the mask:

imout = im.point(lambda i: expression and 255)
Python only evaluates the portion of a logical expression as is necessary to determine the outcome, and returns the last value examined as the result of the expression. So if the expression above is false (0), Python does not look at the second operand, and thus returns 0. Otherwise, it returns 255.

Enhancement
For more advanced image enhancement, you can use the classes in the ImageEnhance module. Once created from an image, an enhancement object can be used to quickly try out different settings.

You can adjust contrast, brightness, color balance and sharpness in this way.

Enhancing images
from PIL import ImageEnhance

enh = ImageEnhance.Contrast(im)
enh.enhance(1.3).show("30% more contrast")
Image sequences
The Python Imaging Library contains some basic support for image sequences (also called animation formats). Supported sequence formats include FLI/FLC, GIF, and a few experimental formats. TIFF files can also contain more than one frame.

When you open a sequence file, PIL automatically loads the first frame in the sequence. You can use the seek and tell methods to move between different frames:

Reading sequences
from PIL import Image

with Image.open("animation.gif") as im:
    im.seek(1)  # skip to the second frame

    try:
        while 1:
            im.seek(im.tell() + 1)
            # do something to im
    except EOFError:
        pass  # end of sequence
As seen in this example, you’ll get an EOFError exception when the sequence ends.

The following class lets you use the for-statement to loop over the sequence:

Using the ImageSequence Iterator class
from PIL import ImageSequence
for frame in ImageSequence.Iterator(im):
    # ...do something to frame...
PostScript printing
The Python Imaging Library includes functions to print images, text and graphics on PostScript printers. Here’s a simple example:

Drawing PostScript
from PIL import Image
from PIL import PSDraw

with Image.open("hopper.ppm") as im:
    title = "hopper"
    box = (1 * 72, 2 * 72, 7 * 72, 10 * 72)  # in points

    ps = PSDraw.PSDraw()  # default is sys.stdout or sys.stdout.buffer
    ps.begin_document(title)

    # draw the image (75 dpi)
    ps.image(box, im, 75)
    ps.rectangle(box)

    # draw title
    ps.setfont("HelveticaNarrow-Bold", 36)
    ps.text((3 * 72, 4 * 72), title)

    ps.end_document()
More on reading images
As described earlier, the open() function of the Image module is used to open an image file. In most cases, you simply pass it the filename as an argument. Image.open() can be used as a context manager:

from PIL import Image
with Image.open("hopper.ppm") as im:
    ...
If everything goes well, the result is an PIL.Image.Image object. Otherwise, an OSError exception is raised.

You can use a file-like object instead of the filename. The object must implement file.read, file.seek and file.tell methods, and be opened in binary mode.

Reading from an open file
from PIL import Image

with open("hopper.ppm", "rb") as fp:
    im = Image.open(fp)
To read an image from binary data, use the BytesIO class:

Reading from binary data
from PIL import Image
import io

im = Image.open(io.BytesIO(buffer))
Note that the library rewinds the file (using seek(0)) before reading the image header. In addition, seek will also be used when the image data is read (by the load method). If the image file is embedded in a larger file, such as a tar file, you can use the ContainerIO or TarIO modules to access it.

Reading from URL
from PIL import Image
from urllib.request import urlopen
url = "https://python-pillow.org/images/pillow-logo.png"
img = Image.open(urlopen(url))
Reading from a tar archive
from PIL import Image, TarIO

fp = TarIO.TarIO("Tests/images/hopper.tar", "hopper.jpg")
im = Image.open(fp)
Batch processing
Operations can be applied to multiple image files. For example, all PNG images in the current directory can be saved as JPEGs at reduced quality.

import glob
from PIL import Image


def compress_image(source_path, dest_path):
    with Image.open(source_path) as img:
        if img.mode != "RGB":
            img = img.convert("RGB")
        img.save(dest_path, "JPEG", optimize=True, quality=80)


paths = glob.glob("*.png")
for path in paths:
    compress_image(path, path[:-4] + ".jpg")
Since images can also be opened from a Path from the pathlib module, the example could be modified to use pathlib instead of the glob module.

from pathlib import Path

paths = Path(".").glob("*.png")
for path in paths:
    compress_image(path, path.stem + ".jpg")
Controlling the decoder
Some decoders allow you to manipulate the image while reading it from a file. This can often be used to speed up decoding when creating thumbnails (when speed is usually more important than quality) and printing to a monochrome laser printer (when only a greyscale version of the image is needed).

The draft() method manipulates an opened but not yet loaded image so it as closely as possible matches the given mode and size. This is done by reconfiguring the image decoder.

Reading in draft mode
This is only available for JPEG and MPO files.

from PIL import Image

with Image.open(file) as im:
    print("original =", im.mode, im.size)

    im.draft("L", (100, 100))
    print("draft =", im.mode, im.size)
This prints something like:

original = RGB (512, 512)
draft = L (128, 128)
Note that the resulting image may not exactly match the requested mode and size. To make sure that the image is not larger than the given size, use the thumbnail method instead.



Concepts
The Python Imaging Library handles raster images; that is, rectangles of pixel data.

Bands
An image can consist of one or more bands of data. The Python Imaging Library allows you to store several bands in a single image, provided they all have the same dimensions and depth. For example, a PNG image might have ‘R’, ‘G’, ‘B’, and ‘A’ bands for the red, green, blue, and alpha transparency values. Many operations act on each band separately, e.g., histograms. It is often useful to think of each pixel as having one value per band.

To get the number and names of bands in an image, use the getbands() method.

Modes
The mode of an image is a string which defines the type and depth of a pixel in the image. Each pixel uses the full range of the bit depth. So a 1-bit pixel has a range of 0-1, an 8-bit pixel has a range of 0-255, a 32-signed integer pixel has the range of INT32 and a 32-bit floating point pixel has the range of FLOAT32. The current release supports the following standard modes:

1 (1-bit pixels, black and white, stored with one pixel per byte)

L (8-bit pixels, black and white)

P (8-bit pixels, mapped to any other mode using a color palette)

RGB (3x8-bit pixels, true color)

RGBA (4x8-bit pixels, true color with transparency mask)

CMYK (4x8-bit pixels, color separation)

YCbCr (3x8-bit pixels, color video format)

Note that this refers to the JPEG, and not the ITU-R BT.2020, standard

LAB (3x8-bit pixels, the L*a*b color space)

HSV (3x8-bit pixels, Hue, Saturation, Value color space)

Hue’s range of 0-255 is a scaled version of 0 degrees <= Hue < 360 degrees

I (32-bit signed integer pixels)

F (32-bit floating point pixels)

Pillow also provides limited support for a few additional modes, including:

LA (L with alpha)

PA (P with alpha)

RGBX (true color with padding)

RGBa (true color with premultiplied alpha)

La (L with premultiplied alpha)

I;16 (16-bit unsigned integer pixels)

I;16L (16-bit little endian unsigned integer pixels)

I;16B (16-bit big endian unsigned integer pixels)

I;16N (16-bit native endian unsigned integer pixels)

BGR;15 (15-bit reversed true colour)

BGR;16 (16-bit reversed true colour)

BGR;24 (24-bit reversed true colour)

BGR;32 (32-bit reversed true colour)

Premultiplied alpha is where the values for each other channel have been multiplied by the alpha. For example, an RGBA pixel of (10, 20, 30, 127) would convert to an RGBa pixel of (5, 10, 15, 127). The values of the R, G and B channels are halved as a result of the half transparency in the alpha channel.

Apart from these additional modes, Pillow doesn’t yet support multichannel images with a depth of more than 8 bits per channel.

Pillow also doesn’t support user-defined modes; if you need to handle band combinations that are not listed above, use a sequence of Image objects.

You can read the mode of an image through the mode attribute. This is a string containing one of the above values.

Size
You can read the image size through the size attribute. This is a 2-tuple, containing the horizontal and vertical size in pixels.

Coordinate System
The Python Imaging Library uses a Cartesian pixel coordinate system, with (0,0) in the upper left corner. Note that the coordinates refer to the implied pixel corners; the centre of a pixel addressed as (0, 0) actually lies at (0.5, 0.5).

Coordinates are usually passed to the library as 2-tuples (x, y). Rectangles are represented as 4-tuples, with the upper left corner given first. For example, a rectangle covering all of an 800x600 pixel image is written as (0, 0, 800, 600).

Palette
The palette mode (P) uses a color palette to define the actual color for each pixel.

Info
You can attach auxiliary information to an image using the info attribute. This is a dictionary object.

How such information is handled when loading and saving image files is up to the file format handler (see the chapter on Image file formats). Most handlers add properties to the info attribute when loading an image, but ignore it when saving images.

Transparency
If an image does not have an alpha band, transparency may be specified in the info attribute with a “transparency” key.

Most of the time, the “transparency” value is a single integer, describing which pixel value is transparent in a “1”, “L”, “I” or “P” mode image. However, PNG images may have three values, one for each channel in an “RGB” mode image, or can have a byte string for a “P” mode image, to specify the alpha value for each palette entry.

Orientation
A common element of the info attribute for JPG and TIFF images is the EXIF orientation tag. This is an instruction for how the image data should be oriented. For example, it may instruct an image to be rotated by 90 degrees, or to be mirrored. To apply this information to an image, exif_transpose() can be used.

Filters
For geometry operations that may map multiple input pixels to a single output pixel, the Python Imaging Library provides different resampling filters.

PIL.Image.NEAREST
Pick one nearest pixel from the input image. Ignore all other input pixels.

PIL.Image.BOX
Each pixel of source image contributes to one pixel of the destination image with identical weights. For upscaling is equivalent of NEAREST. This filter can only be used with the resize() and thumbnail() methods.

New in version 3.4.0.

PIL.Image.BILINEAR
For resize calculate the output pixel value using linear interpolation on all pixels that may contribute to the output value. For other transformations linear interpolation over a 2x2 environment in the input image is used.

PIL.Image.HAMMING
Produces a sharper image than BILINEAR, doesn’t have dislocations on local level like with BOX. This filter can only be used with the resize() and thumbnail() methods.

New in version 3.4.0.

PIL.Image.BICUBIC
For resize calculate the output pixel value using cubic interpolation on all pixels that may contribute to the output value. For other transformations cubic interpolation over a 4x4 environment in the input image is used.

PIL.Image.LANCZOS
Calculate the output pixel value using a high-quality Lanczos filter (a truncated sinc) on all pixels that may contribute to the output value. This filter can only be used with the resize() and thumbnail() methods.

New in version 1.1.3.

Filters comparison table
Filter

Downscaling quality

Upscaling quality

Performance

NEAREST

⭐⭐⭐⭐⭐

BOX

⭐

⭐⭐⭐⭐

BILINEAR

⭐

⭐

⭐⭐⭐

HAMMING

⭐⭐

⭐⭐⭐

BICUBIC

⭐⭐⭐

⭐⭐⭐

⭐⭐

LANCZOS

⭐⭐⭐⭐

⭐⭐⭐⭐

⭐





Image file formats
The Python Imaging Library supports a wide variety of raster file formats. Over 30 different file formats can be identified and read by the library. Write support is less extensive, but most common interchange and presentation formats are supported.

The open() function identifies files from their contents, not their names, but the save() method looks at the name to determine which format to use, unless the format is given explicitly.

When an image is opened from a file, only that instance of the image is considered to have the format. Copies of the image will contain data loaded from the file, but not the file itself, meaning that it can no longer be considered to be in the original format. So if copy() is called on an image, or another method internally creates a copy of the image, then any methods or attributes specific to the format will no longer be present. The fp (file pointer) attribute will no longer be present, and the format attribute will be None.

Fully supported formats
BLP
BLP is the Blizzard Mipmap Format, a texture format used in World of Warcraft. Pillow supports reading JPEG Compressed or raw BLP1 images, and all types of BLP2 images.

Saving
Pillow supports writing BLP images. The save() method can take the following keyword arguments:

blp_version
If present and set to “BLP1”, images will be saved as BLP1. Otherwise, images will be saved as BLP2.

BMP
Pillow reads and writes Windows and OS/2 BMP files containing 1, L, P, or RGB data. 16-colour images are read as P images. Support for reading 8-bit run-length encoding was added in Pillow 9.1.0. Support for reading 4-bit run-length encoding was added in Pillow 9.3.0.

Opening
The open() method sets the following info properties:

compression
Set to 1 if the file is a 256-color run-length encoded image. Set to 2 if the file is a 16-color run-length encoded image.

DDS
DDS is a popular container texture format used in video games and natively supported by DirectX. Uncompressed RGB and RGBA can be read, and (since 8.3.0) written. DXT1, DXT3 (since 3.4.0) and DXT5 pixel formats can be read, only in RGBA mode.

DIB
Pillow reads and writes DIB files. DIB files are similar to BMP files, so see above for more information.

New in version 6.0.0.

EPS
Pillow identifies EPS files containing image data, and can read files that contain embedded raster images (ImageData descriptors). If Ghostscript is available, other EPS files can be read as well. The EPS driver can also write EPS images. The EPS driver can read EPS images in L, LAB, RGB and CMYK mode, but Ghostscript may convert the images to RGB mode rather than leaving them in the original color space. The EPS driver can write images in L, RGB and CMYK modes.

Loading
If Ghostscript is available, you can call the load() method with the following parameters to affect how Ghostscript renders the EPS

scale
Affects the scale of the resultant rasterized image. If the EPS suggests that the image be rendered at 100px x 100px, setting this parameter to 2 will make the Ghostscript render a 200px x 200px image instead. The relative position of the bounding box is maintained:

im = Image.open(...)
im.size  # (100,100)
im.load(scale=2)
im.size  # (200,200)
transparency
If true, generates an RGBA image with a transparent background, instead of the default behaviour of an RGB image with a white background.

GIF
Pillow reads GIF87a and GIF89a versions of the GIF file format. The library writes files in GIF87a by default, unless GIF89a features are used or GIF89a is already in use. Files are written with LZW encoding.

GIF files are initially read as grayscale (L) or palette mode (P) images. Seeking to later frames in a P image will change the image to RGB (or RGBA if the first frame had transparency).

P mode images are changed to RGB because each frame of a GIF may contain its own individual palette of up to 256 colors. When a new frame is placed onto a previous frame, those colors may combine to exceed the P mode limit of 256 colors. Instead, the image is converted to RGB handle this.

If you would prefer the first P image frame to be RGB as well, so that every P frame is converted to RGB or RGBA mode, there is a setting available:

from PIL import GifImagePlugin
GifImagePlugin.LOADING_STRATEGY = GifImagePlugin.LoadingStrategy.RGB_ALWAYS
GIF frames do not always contain individual palettes however. If there is only a global palette, then all of the colors can fit within P mode. If you would prefer the frames to be kept as P in that case, there is also a setting available:

from PIL import GifImagePlugin
GifImagePlugin.LOADING_STRATEGY = GifImagePlugin.LoadingStrategy.RGB_AFTER_DIFFERENT_PALETTE_ONLY
To restore the default behavior, where P mode images are only converted to RGB or RGBA after the first frame:

from PIL import GifImagePlugin
GifImagePlugin.LOADING_STRATEGY = GifImagePlugin.LoadingStrategy.RGB_AFTER_FIRST
Opening
The open() method sets the following info properties:

background
Default background color (a palette color index).

transparency
Transparency color index. This key is omitted if the image is not transparent.

version
Version (either GIF87a or GIF89a).

duration
May not be present. The time to display the current frame of the GIF, in milliseconds.

loop
May not be present. The number of times the GIF should loop. 0 means that it will loop forever.

comment
May not be present. A comment about the image. This is the last comment found before the current frame’s image.

extension
May not be present. Contains application specific information.

Reading sequences
The GIF loader supports the seek() and tell() methods. You can combine these methods to seek to the next frame (im.seek(im.tell() + 1)).

im.seek() raises an EOFError if you try to seek after the last frame.

Saving
When calling save() to write a GIF file, the following options are available:

im.save(out, save_all=True, append_images=[im1, im2, ...])
save_all
If present and true, all frames of the image will be saved. If not, then only the first frame of a multiframe image will be saved.

append_images
A list of images to append as additional frames. Each of the images in the list can be single or multiframe images. This is currently supported for GIF, PDF, PNG, TIFF, and WebP.

It is also supported for ICO and ICNS. If images are passed in of relevant sizes, they will be used instead of scaling down the main image.

include_color_table
Whether or not to include local color table.

interlace
Whether or not the image is interlaced. By default, it is, unless the image is less than 16 pixels in width or height.

disposal
Indicates the way in which the graphic is to be treated after being displayed.

0 - No disposal specified.

1 - Do not dispose.

2 - Restore to background color.

3 - Restore to previous content.

Pass a single integer for a constant disposal, or a list or tuple to set the disposal for each frame separately.

palette
Use the specified palette for the saved image. The palette should be a bytes or bytearray object containing the palette entries in RGBRGB… form. It should be no more than 768 bytes. Alternately, the palette can be passed in as an PIL.ImagePalette.ImagePalette object.

optimize
If present and true, attempt to compress the palette by eliminating unused colors. This is only useful if the palette can be compressed to the next smaller power of 2 elements.

Note that if the image you are saving comes from an existing GIF, it may have the following properties in its info dictionary. For these options, if you do not pass them in, they will default to their info values.

transparency
Transparency color index.

duration
The display duration of each frame of the multiframe gif, in milliseconds. Pass a single integer for a constant duration, or a list or tuple to set the duration for each frame separately.

loop
Integer number of times the GIF should loop. 0 means that it will loop forever. By default, the image will not loop.

comment
A comment about the image.

Reading local images
The GIF loader creates an image memory the same size as the GIF file’s logical screen size, and pastes the actual pixel data (the local image) into this image. If you only want the actual pixel rectangle, you can crop the image:

im = Image.open(...)

if im.tile[0][0] == "gif":
    # only read the first "local image" from this GIF file
    box = im.tile[0][1]
    im = im.crop(box)
ICNS
Pillow reads and writes macOS .icns files. By default, the largest available icon is read, though you can override this by setting the size property before calling load(). The open() method sets the following info property:

Note

Prior to version 8.3.0, Pillow could only write ICNS files on macOS.

sizes
A list of supported sizes found in this icon file; these are a 3-tuple, (width, height, scale), where scale is 2 for a retina icon and 1 for a standard icon. You are permitted to use this 3-tuple format for the size property if you set it before calling load(); after loading, the size will be reset to a 2-tuple containing pixel dimensions (so, e.g. if you ask for (512, 512, 2), the final value of size will be (1024, 1024)).

Saving
The save() method can take the following keyword arguments:

append_images
A list of images to replace the scaled down versions of the image. The order of the images does not matter, as their use is determined by the size of each image.

New in version 5.1.0.

ICO
ICO is used to store icons on Windows. The largest available icon is read.

Saving
The save() method supports the following options:

sizes
A list of sizes including in this ico file; these are a 2-tuple, (width, height); Default to [(16, 16), (24, 24), (32, 32), (48, 48), (64, 64), (128, 128), (256, 256)]. Any sizes bigger than the original size or 256 will be ignored.

The save() method can take the following keyword arguments:

append_images
A list of images to replace the scaled down versions of the image. The order of the images does not matter, as their use is determined by the size of each image.

New in version 8.1.0.

bitmap_format
By default, the image data will be saved in PNG format. With a bitmap format of “bmp”, image data will be saved in BMP format instead.

New in version 8.3.0.

IM
IM is a format used by LabEye and other applications based on the IFUNC image processing library. The library reads and writes most uncompressed interchange versions of this format.

IM is the only format that can store all internal Pillow formats.

JPEG
Pillow reads JPEG, JFIF, and Adobe JPEG files containing L, RGB, or CMYK data. It writes standard and progressive JFIF files.

Using the draft() method, you can speed things up by converting RGB images to L, and resize images to 1/2, 1/4 or 1/8 of their original size while loading them.

By default Pillow doesn’t allow loading of truncated JPEG files, set ImageFile.LOAD_TRUNCATED_IMAGES to override this.

Opening
The open() method may set the following info properties if available:

jfif
JFIF application marker found. If the file is not a JFIF file, this key is not present.

jfif_version
A tuple representing the jfif version, (major version, minor version).

jfif_density
A tuple representing the pixel density of the image, in units specified by jfif_unit.

jfif_unit
Units for the jfif_density:

0 - No Units

1 - Pixels per Inch

2 - Pixels per Centimeter

dpi
A tuple representing the reported pixel density in pixels per inch, if the file is a jfif file and the units are in inches.

adobe
Adobe application marker found. If the file is not an Adobe JPEG file, this key is not present.

adobe_transform
Vendor Specific Tag.

progression
Indicates that this is a progressive JPEG file.

icc_profile
The ICC color profile for the image.

exif
Raw EXIF data from the image.

comment
A comment about the image.

New in version 7.1.0.

Saving
The save() method supports the following options:

quality
The image quality, on a scale from 0 (worst) to 95 (best), or the string keep. The default is 75. Values above 95 should be avoided; 100 disables portions of the JPEG compression algorithm, and results in large files with hardly any gain in image quality. The value keep is only valid for JPEG files and will retain the original image quality level, subsampling, and qtables.

optimize
If present and true, indicates that the encoder should make an extra pass over the image in order to select optimal encoder settings.

progressive
If present and true, indicates that this image should be stored as a progressive JPEG file.

dpi
A tuple of integers representing the pixel density, (x,y).

icc_profile
If present and true, the image is stored with the provided ICC profile. If this parameter is not provided, the image will be saved with no profile attached. To preserve the existing profile:

im.save(filename, 'jpeg', icc_profile=im.info.get('icc_profile'))
exif
If present, the image will be stored with the provided raw EXIF data.

subsampling
If present, sets the subsampling for the encoder.

keep: Only valid for JPEG files, will retain the original image setting.

4:4:4, 4:2:2, 4:2:0: Specific sampling values

0: equivalent to 4:4:4

1: equivalent to 4:2:2

2: equivalent to 4:2:0

If absent, the setting will be determined by libjpeg or libjpeg-turbo.

qtables
If present, sets the qtables for the encoder. This is listed as an advanced option for wizards in the JPEG documentation. Use with caution. qtables can be one of several types of values:

a string, naming a preset, e.g. keep, web_low, or web_high

a list, tuple, or dictionary (with integer keys = range(len(keys))) of lists of 64 integers. There must be between 2 and 4 tables.

New in version 2.5.0.

comment
A comment about the image.

New in version 9.4.0.

Note

To enable JPEG support, you need to build and install the IJG JPEG library before building the Python Imaging Library. See the distribution README for details.

JPEG 2000
New in version 2.4.0.

Pillow reads and writes JPEG 2000 files containing L, LA, RGB or RGBA data. It can also read files containing YCbCr data, which it converts on read into RGB or RGBA depending on whether or not there is an alpha channel. Pillow supports JPEG 2000 raw codestreams (.j2k files), as well as boxed JPEG 2000 files (.j2p or .jpx files). Pillow does not support files whose components have different sampling frequencies.

When loading, if you set the mode on the image prior to the load() method being invoked, you can ask Pillow to convert the image to either RGB or RGBA rather than choosing for itself. It is also possible to set reduce to the number of resolutions to discard (each one reduces the size of the resulting image by a factor of 2), and layers to specify the number of quality layers to load.

Saving
The save() method supports the following options:

offset
The image offset, as a tuple of integers, e.g. (16, 16)

tile_offset
The tile offset, again as a 2-tuple of integers.

tile_size
The tile size as a 2-tuple. If not specified, or if set to None, the image will be saved without tiling.

quality_mode
Either "rates" or "dB" depending on the units you want to use to specify image quality.

quality_layers
A sequence of numbers, each of which represents either an approximate size reduction (if quality mode is "rates") or a signal to noise ratio value in decibels. If not specified, defaults to a single layer of full quality.

num_resolutions
The number of different image resolutions to be stored (which corresponds to the number of Discrete Wavelet Transform decompositions plus one).

codeblock_size
The code-block size as a 2-tuple. Minimum size is 4 x 4, maximum is 1024 x 1024, with the additional restriction that no code-block may have more than 4096 coefficients (i.e. the product of the two numbers must be no greater than 4096).

precinct_size
The precinct size as a 2-tuple. Must be a power of two along both axes, and must be greater than the code-block size.

irreversible
If True, use the lossy discrete waveform transformation DWT 9-7. Defaults to False, which uses the lossless DWT 5-3.

mct
If 1 then enable multiple component transformation when encoding, otherwise use 0 for no component transformation (default). If MCT is enabled and irreversible is True then the Irreversible Color Transformation will be applied, otherwise encoding will use the Reversible Color Transformation. MCT works best with a mode of RGB and is only applicable when the image data has 3 components.

New in version 9.1.0.

progression
Controls the progression order; must be one of "LRCP", "RLCP", "RPCL", "PCRL", "CPRL". The letters stand for Component, Position, Resolution and Layer respectively and control the order of encoding, the idea being that e.g. an image encoded using LRCP mode can have its quality layers decoded as they arrive at the decoder, while one encoded using RLCP mode will have increasing resolutions decoded as they arrive, and so on.

signed
If true, then tell the encoder to save the image as signed.

New in version 9.4.0.

cinema_mode
Set the encoder to produce output compliant with the digital cinema specifications. The options here are "no" (the default), "cinema2k-24" for 24fps 2K, "cinema2k-48" for 48fps 2K, and "cinema4k-24" for 24fps 4K. Note that for compliant 2K files, at least one of your image dimensions must match 2048 x 1080, while for compliant 4K files, at least one of the dimensions must match 4096 x 2160.

no_jp2
If True then don’t wrap the raw codestream in the JP2 file format when saving, otherwise the extension of the filename will be used to determine the format (default).

New in version 9.1.0.

Note

To enable JPEG 2000 support, you need to build and install the OpenJPEG library, version 2.0.0 or higher, before building the Python Imaging Library.

Windows users can install the OpenJPEG binaries available on the OpenJPEG website, but must add them to their PATH in order to use Pillow (if you fail to do this, you will get errors about not being able to load the _imaging DLL).

MSP
Pillow identifies and reads MSP files from Windows 1 and 2. The library writes uncompressed (Windows 1) versions of this format.

PCX
Pillow reads and writes PCX files containing 1, L, P, or RGB data.

PNG
Pillow identifies, reads, and writes PNG files containing 1, L, LA, I, P, RGB or RGBA data. Interlaced files are supported as of v1.1.7.

As of Pillow 6.0, EXIF data can be read from PNG images. However, unlike other image formats, EXIF data is not guaranteed to be present in info until load() has been called.

By default Pillow doesn’t allow loading of truncated PNG files, set ImageFile.LOAD_TRUNCATED_IMAGES to override this.

Opening
The open() function sets the following info properties, when appropriate:

chromaticity
The chromaticity points, as an 8 tuple of floats. (White Point X, White Point Y, Red X, Red Y, Green X, Green Y, Blue X, Blue Y)

gamma
Gamma, given as a floating point number.

srgb
The sRGB rendering intent as an integer.

0 Perceptual

1 Relative Colorimetric

2 Saturation

3 Absolute Colorimetric

transparency
For P images: Either the palette index for full transparent pixels, or a byte string with alpha values for each palette entry.

For 1, L, I and RGB images, the color that represents full transparent pixels in this image.

This key is omitted if the image is not a transparent palette image.

open also sets Image.text to a dictionary of the values of the tEXt, zTXt, and iTXt chunks of the PNG image. Individual compressed chunks are limited to a decompressed size of PngImagePlugin.MAX_TEXT_CHUNK, by default 1MB, to prevent decompression bombs. Additionally, the total size of all of the text chunks is limited to PngImagePlugin.MAX_TEXT_MEMORY, defaulting to 64MB.

Saving
The save() method supports the following options:

optimize
If present and true, instructs the PNG writer to make the output file as small as possible. This includes extra processing in order to find optimal encoder settings.

transparency
For P, 1, L, I, and RGB images, this option controls what color from the image to mark as transparent.

For P images, this can be a either the palette index, or a byte string with alpha values for each palette entry.

dpi
A tuple of two numbers corresponding to the desired dpi in each direction.

pnginfo
A PIL.PngImagePlugin.PngInfo instance containing chunks.

compress_level
ZLIB compression level, a number between 0 and 9: 1 gives best speed, 9 gives best compression, 0 gives no compression at all. Default is 6. When optimize option is True compress_level has no effect (it is set to 9 regardless of a value passed).

icc_profile
The ICC Profile to include in the saved file.

exif
The exif data to include in the saved file.

New in version 6.0.0.

bits (experimental)
For P images, this option controls how many bits to store. If omitted, the PNG writer uses 8 bits (256 colors).

dictionary (experimental)
Set the ZLIB encoder dictionary.

Note

To enable PNG support, you need to build and install the ZLIB compression library before building the Python Imaging Library. See the installation documentation for details.

APNG sequences
The PNG loader includes limited support for reading and writing Animated Portable Network Graphics (APNG) files. When an APNG file is loaded, get_format_mimetype() will return "image/apng". The value of the is_animated property will be True when the n_frames property is greater than 1. For APNG files, the n_frames property depends on both the animation frame count as well as the presence or absence of a default image. See the default_image property documentation below for more details. The seek() and tell() methods are supported.

im.seek() raises an EOFError if you try to seek after the last frame.

These info properties will be set for APNG frames, where applicable:

default_image
Specifies whether or not this APNG file contains a separate default image, which is not a part of the actual APNG animation.

When an APNG file contains a default image, the initially loaded image (i.e. the result of seek(0)) will be the default image. To account for the presence of the default image, the n_frames property will be set to frame_count + 1, where frame_count is the actual APNG animation frame count. To load the first APNG animation frame, seek(1) must be called.

True - The APNG contains default image, which is not an animation frame.

False - The APNG does not contain a default image. The n_frames property will be set to the actual APNG animation frame count. The initially loaded image (i.e. seek(0)) will be the first APNG animation frame.

loop
The number of times to loop this APNG, 0 indicates infinite looping.

duration
The time to display this APNG frame (in milliseconds).

Note

The APNG loader returns images the same size as the APNG file’s logical screen size. The returned image contains the pixel data for a given frame, after applying any APNG frame disposal and frame blend operations (i.e. it contains what a web browser would render for this frame - the composite of all previous frames and this frame).

Any APNG file containing sequence errors is treated as an invalid image. The APNG loader will not attempt to repair and reorder files containing sequence errors.

Saving
When calling save(), by default only a single frame PNG file will be saved. To save an APNG file (including a single frame APNG), the save_all parameter must be set to True. The following parameters can also be set:

default_image
Boolean value, specifying whether or not the base image is a default image. If True, the base image will be used as the default image, and the first image from the append_images sequence will be the first APNG animation frame. If False, the base image will be used as the first APNG animation frame. Defaults to False.

append_images
A list or tuple of images to append as additional frames. Each of the images in the list can be single or multiframe images. The size of each frame should match the size of the base image. Also note that if a frame’s mode does not match that of the base image, the frame will be converted to the base image mode.

loop
Integer number of times to loop this APNG, 0 indicates infinite looping. Defaults to 0.

duration
Integer (or list or tuple of integers) length of time to display this APNG frame (in milliseconds). Defaults to 0.

disposal
An integer (or list or tuple of integers) specifying the APNG disposal operation to be used for this frame before rendering the next frame. Defaults to 0.

0 (OP_NONE, default) - No disposal is done on this frame before rendering the next frame.

1 (PIL.PngImagePlugin.Disposal.OP_BACKGROUND) - This frame’s modified region is cleared to fully transparent black before rendering the next frame.

2 (OP_PREVIOUS) - This frame’s modified region is reverted to the previous frame’s contents before rendering the next frame.

blend
An integer (or list or tuple of integers) specifying the APNG blend operation to be used for this frame before rendering the next frame. Defaults to 0.

0 (OP_SOURCE) - All color components of this frame, including alpha, overwrite the previous output image contents.

1 (OP_OVER) - This frame should be alpha composited with the previous output image contents.

Note

The duration, disposal and blend parameters can be set to lists or tuples to specify values for each individual frame in the animation. The length of the list or tuple must be identical to the total number of actual frames in the APNG animation. If the APNG contains a default image (i.e. default_image is set to True), these list or tuple parameters should not include an entry for the default image.

PPM
Pillow reads and writes PBM, PGM, PPM and PNM files containing 1, L, I or RGB data.

SGI
Pillow reads and writes uncompressed L, RGB, and RGBA files.

SPIDER
Pillow reads and writes SPIDER image files of 32-bit floating point data (“F;32F”).

Pillow also reads SPIDER stack files containing sequences of SPIDER images. The seek() and tell() methods are supported, and random access is allowed.

Opening
The open() method sets the following attributes:

format
Set to SPIDER

istack
Set to 1 if the file is an image stack, else 0.

n_frames
Set to the number of images in the stack.

A convenience method, convert2byte(), is provided for converting floating point data to byte data (mode L):

im = Image.open("image001.spi").convert2byte()
Saving
The extension of SPIDER files may be any 3 alphanumeric characters. Therefore the output format must be specified explicitly:

im.save('newimage.spi', format='SPIDER')
For more information about the SPIDER image processing package, see https://github.com/spider-em/SPIDER

TGA
Pillow reads and writes TGA images containing L, LA, P, RGB, and RGBA data. Pillow can read and write both uncompressed and run-length encoded TGAs.

Saving
The save() method can take the following keyword arguments:

compression
If set to “tga_rle”, the file will be run-length encoded.

New in version 5.3.0.

id_section
The identification field.

New in version 5.3.0.

orientation
If present and a positive number, the first pixel is for the top left corner, rather than the bottom left corner.

New in version 5.3.0.

TIFF
Pillow reads and writes TIFF files. It can read both striped and tiled images, pixel and plane interleaved multi-band images. If you have libtiff and its headers installed, Pillow can read and write many kinds of compressed TIFF files. If not, Pillow will only read and write uncompressed files.

Note

Beginning in version 5.0.0, Pillow requires libtiff to read or write compressed files. Prior to that release, Pillow had buggy support for reading Packbits, LZW and JPEG compressed TIFFs without using libtiff.

Opening
The open() method sets the following info properties:

compression
Compression mode.

New in version 2.0.0.

dpi
Image resolution as an (xdpi, ydpi) tuple, where applicable. You can use the tag attribute to get more detailed information about the image resolution.

New in version 1.1.5.

resolution
Image resolution as an (xres, yres) tuple, where applicable. This is a measurement in whichever unit is specified by the file.

New in version 1.1.5.

The tag_v2 attribute contains a dictionary of TIFF metadata. The keys are numerical indexes from TiffTags.TAGS_V2. Values are strings or numbers for single items, multiple values are returned in a tuple of values. Rational numbers are returned as a IFDRational object.

New in version 3.0.0.

For compatibility with legacy code, the tag attribute contains a dictionary of decoded TIFF fields as returned prior to version 3.0.0. Values are returned as either strings or tuples of numeric values. Rational numbers are returned as a tuple of (numerator, denominator).

Deprecated since version 3.0.0.

Reading Multi-frame TIFF Images
The TIFF loader supports the seek() and tell() methods, taking and returning frame numbers within the image file. You can combine these methods to seek to the next frame (im.seek(im.tell() + 1)). Frames are numbered from 0 to im.n_frames - 1, and can be accessed in any order.

im.seek() raises an EOFError if you try to seek after the last frame.

Saving
The save() method can take the following keyword arguments:

save_all
If true, Pillow will save all frames of the image to a multiframe tiff document.

New in version 3.4.0.

append_images
A list of images to append as additional frames. Each of the images in the list can be single or multiframe images. Note however, that for correct results, all the appended images should have the same encoderinfo and encoderconfig properties.

New in version 4.2.0.

tiffinfo
A ImageFileDirectory_v2 object or dict object containing tiff tags and values. The TIFF field type is autodetected for Numeric and string values, any other types require using an ImageFileDirectory_v2 object and setting the type in tagtype with the appropriate numerical value from TiffTags.TYPES.

New in version 2.3.0.

Metadata values that are of the rational type should be passed in using a IFDRational object.

New in version 3.1.0.

For compatibility with legacy code, a ImageFileDirectory_v1 object may be passed in this field. However, this is deprecated.

New in version 5.4.0.

Previous versions only supported some tags when writing using libtiff. The supported list is found in TiffTags.LIBTIFF_CORE.

New in version 6.1.0.

Added support for signed types (e.g. TIFF_SIGNED_LONG) and multiple values. Multiple values for a single tag must be to ImageFileDirectory_v2 as a tuple and require a matching type in tagtype tagtype.

exif
Alternate keyword to “tiffinfo”, for consistency with other formats.

New in version 8.4.0.

compression
A string containing the desired compression method for the file. (valid only with libtiff installed) Valid compression methods are: None, "group3", "group4", "jpeg", "lzma", "packbits", "tiff_adobe_deflate", "tiff_ccitt", "tiff_lzw", "tiff_raw_16", "tiff_sgilog", "tiff_sgilog24", "tiff_thunderscan", "webp", "zstd"

quality
The image quality for JPEG compression, on a scale from 0 (worst) to 100 (best). The default is 75.

New in version 6.1.0.

These arguments to set the tiff header fields are an alternative to using the general tags available through tiffinfo.

description

software

date_time

artist

copyright
Strings

icc_profile
The ICC Profile to include in the saved file.

resolution_unit
An integer. 1 for no unit, 2 for inches and 3 for centimeters.

resolution
Either an integer or a float, used for both the x and y resolution.

x_resolution
Either an integer or a float.

y_resolution
Either an integer or a float.

dpi
A tuple of (x_resolution, y_resolution), with inches as the resolution unit. For consistency with other image formats, the x and y resolutions of the dpi will be rounded to the nearest integer.

WebP
Pillow reads and writes WebP files. The specifics of Pillow’s capabilities with this format are currently undocumented.

Saving
The save() method supports the following options:

lossless
If present and true, instructs the WebP writer to use lossless compression.

quality
Integer, 1-100, Defaults to 80. For lossy, 0 gives the smallest size and 100 the largest. For lossless, this parameter is the amount of effort put into the compression: 0 is the fastest, but gives larger files compared to the slowest, but best, 100.

method
Quality/speed trade-off (0=fast, 6=slower-better). Defaults to 4.

exact
If true, preserve the transparent RGB values. Otherwise, discard invisible RGB values for better compression. Defaults to false. Requires libwebp 0.5.0 or later.

icc_profile
The ICC Profile to include in the saved file. Only supported if the system WebP library was built with webpmux support.

exif
The exif data to include in the saved file. Only supported if the system WebP library was built with webpmux support.

Saving sequences
Note

Support for animated WebP files will only be enabled if the system WebP library is v0.5.0 or later. You can check webp animation support at runtime by calling features.check("webp_anim").

When calling save() to write a WebP file, by default only the first frame of a multiframe image will be saved. If the save_all argument is present and true, then all frames will be saved, and the following options will also be available.

append_images
A list of images to append as additional frames. Each of the images in the list can be single or multiframe images.

duration
The display duration of each frame, in milliseconds. Pass a single integer for a constant duration, or a list or tuple to set the duration for each frame separately.

loop
Number of times to repeat the animation. Defaults to [0 = infinite].

background
Background color of the canvas, as an RGBA tuple with values in the range of (0-255).

minimize_size
If true, minimize the output size (slow). Implicitly disables key-frame insertion.

kmin, kmax
Minimum and maximum distance between consecutive key frames in the output. The library may insert some key frames as needed to satisfy this criteria. Note that these conditions should hold: kmax > kmin and kmin >= kmax / 2 + 1. Also, if kmax <= 0, then key-frame insertion is disabled; and if kmax == 1, then all frames will be key-frames (kmin value does not matter for these special cases).

allow_mixed
If true, use mixed compression mode; the encoder heuristically chooses between lossy and lossless for each frame.

XBM
Pillow reads and writes X bitmap files (mode 1).

Read-only formats
CUR
CUR is used to store cursors on Windows. The CUR decoder reads the largest available cursor. Animated cursors are not supported.

DCX
DCX is a container file format for PCX files, defined by Intel. The DCX format is commonly used in fax applications. The DCX decoder can read files containing 1, L, P, or RGB data.

When the file is opened, only the first image is read. You can use seek() or ImageSequence to read other images.

FITS
New in version 9.1.0.

Pillow identifies and reads FITS files, commonly used for astronomy.

FLI, FLC
Pillow reads Autodesk FLI and FLC animations.

The open() method sets the following info properties:

duration
The delay (in milliseconds) between each frame.

FPX
Pillow reads Kodak FlashPix files. In the current version, only the highest resolution image is read from the file, and the viewing transform is not taken into account.

Note

To enable full FlashPix support, you need to build and install the IJG JPEG library before building the Python Imaging Library. See the distribution README for details.

FTEX
New in version 3.2.0.

The FTEX decoder reads textures used for 3D objects in Independence War 2: Edge Of Chaos. The plugin reads a single texture per file, in the compressed and uncompressed formats.

GBR
The GBR decoder reads GIMP brush files, version 1 and 2.

Opening
The open() method sets the following info properties:

comment
The brush name.

spacing
The spacing between the brushes, in pixels. Version 2 only.

GD
Pillow reads uncompressed GD2 files. Note that you must use PIL.GdImageFile.open() to read such a file.

Opening
The open() method sets the following info properties:

transparency
Transparency color index. This key is omitted if the image is not transparent.

IMT
Pillow reads Image Tools images containing L data.

IPTC/NAA
Pillow provides limited read support for IPTC/NAA newsphoto files.

MCIDAS
Pillow identifies and reads 8-bit McIdas area files.

MIC
Pillow identifies and reads Microsoft Image Composer (MIC) files. When opened, the first sprite in the file is loaded. You can use seek() and tell() to read other sprites from the file.

Note that there may be an embedded gamma of 2.2 in MIC files.

MPO
Pillow identifies and reads Multi Picture Object (MPO) files, loading the primary image when first opened. The seek() and tell() methods may be used to read other pictures from the file. The pictures are zero-indexed and random access is supported.

Saving
When calling save() to write an MPO file, by default only the first frame of a multiframe image will be saved. If the save_all argument is present and true, then all frames will be saved, and the following option will also be available.

append_images
A list of images to append as additional pictures. Each of the images in the list can be single or multiframe images.

New in version 9.3.0.

PCD
Pillow reads PhotoCD files containing RGB data. This only reads the 768x512 resolution image from the file. Higher resolutions are encoded in a proprietary encoding.

PIXAR
Pillow provides limited support for PIXAR raster files. The library can identify and read “dumped” RGB files.

The format code is PIXAR.

PSD
Pillow identifies and reads PSD files written by Adobe Photoshop 2.5 and 3.0.

SUN
Pillow identifies and reads Sun raster files.

WAL
New in version 1.1.4.

Pillow reads Quake2 WAL texture files.

Note that this file format cannot be automatically identified, so you must use the open function in the WalImageFile module to read files in this format.

By default, a Quake2 standard palette is attached to the texture. To override the palette, use the putpalette method.

WMF, EMF
Pillow can identify WMF and EMF files.

On Windows, it can read WMF and EMF files. By default, it will load the image at 72 dpi. To load it at another resolution:

from PIL import Image

with Image.open("drawing.wmf") as im:
    im.load(dpi=144)
To add other read or write support, use PIL.WmfImagePlugin.register_handler() to register a WMF and EMF handler.

from PIL import Image
from PIL import WmfImagePlugin


class WmfHandler:
    def open(self, im):
        ...

    def load(self, im):
        ...
        return image

    def save(self, im, fp, filename):
        ...


wmf_handler = WmfHandler()

WmfImagePlugin.register_handler(wmf_handler)

im = Image.open("sample.wmf")
XPM
Pillow reads X pixmap files (mode P) with 256 colors or less.

Opening
The open() method sets the following info properties:

transparency
Transparency color index. This key is omitted if the image is not transparent.

Write-only formats
PALM
Pillow provides write-only support for PALM pixmap files.

The format code is Palm, the extension is .palm.

PDF
Pillow can write PDF (Acrobat) images. Such images are written as binary PDF 1.4 files, using either JPEG or HEX encoding depending on the image mode (and whether JPEG support is available or not).

Saving
The save() method can take the following keyword arguments:

save_all
If a multiframe image is used, by default, only the first image will be saved. To save all frames, each frame to a separate page of the PDF, the save_all parameter must be present and set to True.

New in version 3.0.0.

append_images
A list of PIL.Image.Image objects to append as additional pages. Each of the images in the list can be single or multiframe images. The save_all parameter must be present and set to True in conjunction with append_images.

New in version 4.2.0.

append
Set to True to append pages to an existing PDF file. If the file doesn’t exist, an OSError will be raised.

New in version 5.1.0.

resolution
Image resolution in DPI. This, together with the number of pixels in the image, will determine the physical dimensions of the page that will be saved in the PDF.

title
The document’s title. If not appending to an existing PDF file, this will default to the filename.

New in version 5.1.0.

author
The name of the person who created the document.

New in version 5.1.0.

subject
The subject of the document.

New in version 5.1.0.

keywords
Keywords associated with the document.

New in version 5.1.0.

creator
If the document was converted to PDF from another format, the name of the conforming product that created the original document from which it was converted.

New in version 5.1.0.

producer
If the document was converted to PDF from another format, the name of the conforming product that converted it to PDF.

New in version 5.1.0.

creationDate
The creation date of the document. If not appending to an existing PDF file, this will default to the current time.

New in version 5.3.0.

modDate
The modification date of the document. If not appending to an existing PDF file, this will default to the current time.

New in version 5.3.0.

XV Thumbnails
Pillow can read XV thumbnail files.

Identify-only formats
BUFR
New in version 1.1.3.

Pillow provides a stub driver for BUFR files.

To add read or write support to your application, use PIL.BufrStubImagePlugin.register_handler().

GRIB
New in version 1.1.5.

Pillow provides a stub driver for GRIB files.

The driver requires the file to start with a GRIB header. If you have files with embedded GRIB data, or files with multiple GRIB fields, your application has to seek to the header before passing the file handle to Pillow.

To add read or write support to your application, use PIL.GribStubImagePlugin.register_handler().

HDF5
New in version 1.1.5.

Pillow provides a stub driver for HDF5 files.

To add read or write support to your application, use PIL.Hdf5StubImagePlugin.register_handler().

MPEG
Pillow identifies MPEG files.




Text anchors
The anchor parameter determines the alignment of drawn text relative to the xy parameter. The default alignment is top left, specifically la (left-ascender) for horizontal text and lt (left-top) for vertical text.

This parameter is only supported by OpenType/TrueType fonts. Other fonts may ignore the parameter and use the default (top left) alignment.

Specifying an anchor
An anchor is specified with a two-character string. The first character is the horizontal alignment, the second character is the vertical alignment. For example, the default value of la for horizontal text means left-ascender aligned text.

When drawing text with PIL.ImageDraw.ImageDraw.text() with a specific anchor, the text will be placed such that the specified anchor point is at the xy coordinates.

For example, in the following image, the text is ms (middle-baseline) aligned, with xy at the intersection of the two lines:

ms (middle-baseline) aligned text.
from PIL import Image, ImageDraw, ImageFont

font = ImageFont.truetype("Tests/fonts/NotoSans-Regular.ttf", 48)
im = Image.new("RGB", (200, 200), "white")
d = ImageDraw.Draw(im)
d.line(((0, 100), (200, 100)), "gray")
d.line(((100, 0), (100, 200)), "gray")
d.text((100, 100), "Quick", fill="black", anchor="ms", font=font)

Quick reference
Horizontal textVertical text
Horizontal anchor alignment
l — left
Anchor is to the left of the text.

For horizontal text this is the origin of the first glyph, as shown in the FreeType tutorial.

m — middle
Anchor is horizontally centered with the text.

For vertical text it is recommended to use s (baseline) alignment instead, as it does not change based on the specific glyphs of the given text.

r — right
Anchor is to the right of the text.

For horizontal text this is the advanced origin of the last glyph, as shown in the FreeType tutorial.

s — baseline (vertical text only)
Anchor is at the baseline (middle) of the text. The exact alignment depends on the font.

For vertical text this is the recommended alignment, as it does not change based on the specific glyphs of the given text (see image for vertical text above).

Vertical anchor alignment
a — ascender / top (horizontal text only)
Anchor is at the ascender line (top) of the first line of text, as defined by the font.

See Font metrics on Wikipedia for more information.

t — top (single-line text only)
Anchor is at the top of the text.

For vertical text this is the origin of the first glyph, as shown in the FreeType tutorial.

For horizontal text it is recommended to use a (ascender) alignment instead, as it does not change based on the specific glyphs of the given text.

m — middle
Anchor is vertically centered with the text.

For horizontal text this is the midpoint of the first ascender line and the last descender line.

s — baseline (horizontal text only)
Anchor is at the baseline (bottom) of the first line of text, only descenders extend below the anchor.

See Font metrics on Wikipedia for more information.

b — bottom (single-line text only)
Anchor is at the bottom of the text.

For vertical text this is the advanced origin of the last glyph, as shown in the FreeType tutorial.

For horizontal text it is recommended to use d (descender) alignment instead, as it does not change based on the specific glyphs of the given text.

d — descender / bottom (horizontal text only)
Anchor is at the descender line (bottom) of the last line of text, as defined by the font.

See Font metrics on Wikipedia for more information.

Examples
The following image shows several examples of anchors for horizontal text. In each section the xy parameter was set to the center shown by the intersection of the two lines.

Text anchor examples



Writing Your Own Image Plugin
Pillow uses a plugin model which allows you to add your own decoders and encoders to the library, without any changes to the library itself. Such plugins usually have names like XxxImagePlugin.py, where Xxx is a unique format name (usually an abbreviation).

Warning

Pillow >= 2.1.0 no longer automatically imports any file in the Python path with a name ending in ImagePlugin.py. You will need to import your image plugin manually.

Pillow decodes files in two stages:

It loops over the available image plugins in the loaded order, and calls the plugin’s _accept function with the first 16 bytes of the file. If the _accept function returns true, the plugin’s _open method is called to set up the image metadata and image tiles. The _open method is not for decoding the actual image data.

When the image data is requested, the ImageFile.load method is called, which sets up a decoder for each tile and feeds the data to it.

An image plugin should contain a format handler derived from the PIL.ImageFile.ImageFile base class. This class should provide an _open method, which reads the file header and sets up at least the mode and size attributes. To be able to load the file, the method must also create a list of tile descriptors, which contain a decoder name, extents of the tile, and any decoder-specific data. The format handler class must be explicitly registered, via a call to the Image module.

Note

For performance reasons, it is important that the _open method quickly rejects files that do not have the appropriate contents.

Example
The following plugin supports a simple format, which has a 128-byte header consisting of the words “SPAM” followed by the width, height, and pixel size in bits. The header fields are separated by spaces. The image data follows directly after the header, and can be either bi-level, greyscale, or 24-bit true color.

SpamImagePlugin.py:

from PIL import Image, ImageFile


def _accept(prefix):
    return prefix[:4] == b"SPAM"


class SpamImageFile(ImageFile.ImageFile):

    format = "SPAM"
    format_description = "Spam raster image"

    def _open(self):

        header = self.fp.read(128).split()

        # size in pixels (width, height)
        self._size = int(header[1]), int(header[2])

        # mode setting
        bits = int(header[3])
        if bits == 1:
            self.mode = "1"
        elif bits == 8:
            self.mode = "L"
        elif bits == 24:
            self.mode = "RGB"
        else:
            msg = "unknown number of bits"
            raise SyntaxError(msg)

        # data descriptor
        self.tile = [("raw", (0, 0) + self.size, 128, (self.mode, 0, 1))]


Image.register_open(SpamImageFile.format, SpamImageFile, _accept)

Image.register_extensions(
    SpamImageFile.format,
    [
        ".spam",
        ".spa",  # DOS version
    ],
)
The format handler must always set the size and mode attributes. If these are not set, the file cannot be opened. To simplify the plugin, the calling code considers exceptions like SyntaxError, KeyError, IndexError, EOFError and struct.error as a failure to identify the file.

Note that the image plugin must be explicitly registered using PIL.Image.register_open(). Although not required, it is also a good idea to register any extensions used by this format.

Once the plugin has been imported, it can be used:

from PIL import Image
import SpamImagePlugin

with Image.open("hopper.spam") as im:
    pass
The tile attribute
To be able to read the file as well as just identifying it, the tile attribute must also be set. This attribute consists of a list of tile descriptors, where each descriptor specifies how data should be loaded to a given region in the image.

In most cases, only a single descriptor is used, covering the full image. PsdImagePlugin.PsdImageFile uses multiple tiles to combine channels within a single layer, given that the channels are stored separately, one after the other.

The tile descriptor is a 4-tuple with the following contents:

(decoder, region, offset, parameters)
The fields are used as follows:

decoder
Specifies which decoder to use. The raw decoder used here supports uncompressed data, in a variety of pixel formats. For more information on this decoder, see the description below.

A list of C decoders can be seen under codecs section of the function array in _imaging.c. Python decoders are registered within the relevant plugins.

region
A 4-tuple specifying where to store data in the image.

offset
Byte offset from the beginning of the file to image data.

parameters
Parameters to the decoder. The contents of this field depends on the decoder specified by the first field in the tile descriptor tuple. If the decoder doesn’t need any parameters, use None for this field.

Note that the tile attribute contains a list of tile descriptors, not just a single descriptor.

Decoders
The raw decoder
The raw decoder is used to read uncompressed data from an image file. It can be used with most uncompressed file formats, such as PPM, BMP, uncompressed TIFF, and many others. To use the raw decoder with the PIL.Image.frombytes() function, use the following syntax:

image = Image.frombytes(
    mode, size, data, "raw",
    raw_mode, stride, orientation
    )
When used in a tile descriptor, the parameter field should look like:

(raw_mode, stride, orientation)
The fields are used as follows:

raw_mode
The pixel layout used in the file, and is used to properly convert data to PIL’s internal layout. For a summary of the available formats, see the table below.

stride
The distance in bytes between two consecutive lines in the image. If 0, the image is assumed to be packed (no padding between lines). If omitted, the stride defaults to 0.

orientation
Whether the first line in the image is the top line on the screen (1), or the bottom line (-1). If omitted, the orientation defaults to 1.

The raw mode field is used to determine how the data should be unpacked to match PIL’s internal pixel layout. PIL supports a large set of raw modes; for a complete list, see the table in the Unpack.c module. The following table describes some commonly used raw modes:

mode

description

1

1-bit bilevel, stored with the leftmost pixel in the most
significant bit. 0 means black, 1 means white.
1;I

1-bit inverted bilevel, stored with the leftmost pixel in the
most significant bit. 0 means white, 1 means black.
1;R

1-bit reversed bilevel, stored with the leftmost pixel in the
least significant bit. 0 means black, 1 means white.
L

8-bit greyscale. 0 means black, 255 means white.

L;I

8-bit inverted greyscale. 0 means white, 255 means black.

P

8-bit palette-mapped image.

RGB

24-bit true colour, stored as (red, green, blue).

BGR

24-bit true colour, stored as (blue, green, red).

RGBX

24-bit true colour, stored as (red, green, blue, pad). The pad
pixels may vary.
RGB;L

24-bit true colour, line interleaved (first all red pixels, then
all green pixels, finally all blue pixels).
Note that for the most common cases, the raw mode is simply the same as the mode.

The Python Imaging Library supports many other decoders, including JPEG, PNG, and PackBits. For details, see the decode.c source file, and the standard plugin implementations provided with the library.

Decoding floating point data
PIL provides some special mechanisms to allow you to load a wide variety of formats into a mode F (floating point) image memory.

You can use the raw decoder to read images where data is packed in any standard machine data type, using one of the following raw modes:

mode

description

F

32-bit native floating point.

F;8

8-bit unsigned integer.

F;8S

8-bit signed integer.

F;16

16-bit little endian unsigned integer.

F;16S

16-bit little endian signed integer.

F;16B

16-bit big endian unsigned integer.

F;16BS

16-bit big endian signed integer.

F;16N

16-bit native unsigned integer.

F;16NS

16-bit native signed integer.

F;32

32-bit little endian unsigned integer.

F;32S

32-bit little endian signed integer.

F;32B

32-bit big endian unsigned integer.

F;32BS

32-bit big endian signed integer.

F;32N

32-bit native unsigned integer.

F;32NS

32-bit native signed integer.

F;32F

32-bit little endian floating point.

F;32BF

32-bit big endian floating point.

F;32NF

32-bit native floating point.

F;64F

64-bit little endian floating point.

F;64BF

64-bit big endian floating point.

F;64NF

64-bit native floating point.

The bit decoder
If the raw decoder cannot handle your format, PIL also provides a special “bit” decoder that can be used to read various packed formats into a floating point image memory.

To use the bit decoder with the PIL.Image.frombytes() function, use the following syntax:

image = Image.frombytes(
    mode, size, data, "bit",
    bits, pad, fill, sign, orientation
    )
When used in a tile descriptor, the parameter field should look like:

(bits, pad, fill, sign, orientation)
The fields are used as follows:

bits
Number of bits per pixel (2-32). No default.

pad
Padding between lines, in bits. This is either 0 if there is no padding, or 8 if lines are padded to full bytes. If omitted, the pad value defaults to 8.

fill
Controls how data are added to, and stored from, the decoder bit buffer.

fill=0
Add bytes to the LSB end of the decoder buffer; store pixels from the MSB end.

fill=1
Add bytes to the MSB end of the decoder buffer; store pixels from the MSB end.

fill=2
Add bytes to the LSB end of the decoder buffer; store pixels from the LSB end.

fill=3
Add bytes to the MSB end of the decoder buffer; store pixels from the LSB end.

If omitted, the fill order defaults to 0.

sign
If non-zero, bit fields are sign extended. If zero or omitted, bit fields are unsigned.

orientation
Whether the first line in the image is the top line on the screen (1), or the bottom line (-1). If omitted, the orientation defaults to 1.

Writing Your Own File Codec in C
There are 3 stages in a file codec’s lifetime:

Setup: Pillow looks for a function in the decoder or encoder registry, falling back to a function named [codecname]_decoder or [codecname]_encoder on the internal core image object. That function is called with the args tuple from the tile.

Transforming: The codec’s decode or encode function is repeatedly called with chunks of image data.

Cleanup: If the codec has registered a cleanup function, it will be called at the end of the transformation process, even if there was an exception raised.

Setup
The current conventions are that the codec setup function is named PyImaging_[codecname]DecoderNew or PyImaging_[codecname]EncoderNew and defined in decode.c or encode.c. The Python binding for it is named [codecname]_decoder or [codecname]_encoder and is set up from within the _imaging.c file in the codecs section of the function array.

The setup function needs to call PyImaging_DecoderNew or PyImaging_EncoderNew and at the very least, set the decode or encode function pointer. The fields of interest in this object are:

decode/encode
Function pointer to the decode or encode function, which has access to im, state, and the buffer of data to be transformed.

cleanup
Function pointer to the cleanup function, has access to state.

im
The target image, will be set by Pillow.

state
An ImagingCodecStateInstance, will be set by Pillow. The context member is an opaque struct that can be used by the codec to store any format specific state or options.

pulls_fd/pushes_fd
If the decoder has pulls_fd or the encoder has pushes_fd set to 1, state->fd will be a pointer to the Python file like object. The codec may use the functions in codec_fd.c to read or write directly with the file like object rather than have the data pushed through a buffer.

New in version 3.3.0.

Transforming
The decode or encode function is called with the target (core) image, the codec state structure, and a buffer of data to be transformed.

It is the codec’s responsibility to pull as much data as possible out of the buffer and return the number of bytes consumed. The next call to the codec will include the previous unconsumed tail. The codec function will be called multiple times as the data processed.

Alternatively, if pulls_fd or pushes_fd is set, then the decode or encode function is called once, with an empty buffer. It is the codec’s responsibility to transform the entire tile in that one call. Using this will provide a codec with more freedom, but that freedom may mean increased memory usage if the entire tile is held in memory at once by the codec.

If an error occurs, set state->errcode and return -1.

Return -1 on success, without setting the errcode.

Cleanup
The cleanup function is called after the codec returns a negative value, or if there is an error. This function should free any allocated memory and release any resources from external libraries.

Writing Your Own File Codec in Python
Python file decoders and encoders should derive from PIL.ImageFile.PyDecoder and PIL.ImageFile.PyEncoder respectively, and should at least override the decode or encode method. They should be registered using PIL.Image.register_decoder() and PIL.Image.register_encoder(). As in the C implementation of the file codecs, there are three stages in the lifetime of a Python-based file codec:

Setup: Pillow looks for the codec in the decoder or encoder registry, then instantiates the class.

Transforming: The instance’s decode method is repeatedly called with a buffer of data to be interpreted, or the encode method is repeatedly called with the size of data to be output.

Alternatively, if the decoder’s _pulls_fd property (or the encoder’s _pushes_fd property) is set to True, then decode and encode will only be called once. In the decoder, self.fd can be used to access the file-like object. Using this will provide a codec with more freedom, but that freedom may mean increased memory usage if entire file is held in memory at once by the codec.

In decode, once the data has been interpreted, set_as_raw can be used to populate the image.

Cleanup: The instance’s cleanup method is called once the transformation is complete. This can be used to clean up any resources used by the codec.

If you set _pulls_fd or _pushes_fd to True however, then you probably chose to perform any cleanup tasks at the end of decode or encode.

For an example PIL.ImageFile.PyDecoder, see DdsImagePlugin. For a plugin that uses both PIL.ImageFile.PyDecoder and PIL.ImageFile.PyEncoder, see BlpImagePlugin




Image Module
The Image module provides a class with the same name which is used to represent a PIL image. The module also provides a number of factory functions, including functions to load images from files, and to create new images.

Examples
Open, rotate, and display an image (using the default viewer)
The following script loads an image, rotates it 45 degrees, and displays it using an external viewer (usually xv on Unix, and the Paint program on Windows).

from PIL import Image
with Image.open("hopper.jpg") as im:
    im.rotate(45).show()
Create thumbnails
The following script creates nice thumbnails of all JPEG images in the current directory preserving aspect ratios with 128x128 max resolution.

from PIL import Image
import glob, os

size = 128, 128

for infile in glob.glob("*.jpg"):
    file, ext = os.path.splitext(infile)
    with Image.open(infile) as im:
        im.thumbnail(size)
        im.save(file + ".thumbnail", "JPEG")
Functions
PIL.Image.open(fp, mode='r', formats=None)[source]
Opens and identifies the given image file.

This is a lazy operation; this function identifies the file, but the file remains open and the actual image data is not read from the file until you try to process the data (or call the load() method). See new(). See File Handling in Pillow.

PARAMETERS:
fp – A filename (string), pathlib.Path object or a file object. The file object must implement file.read, file.seek, and file.tell methods, and be opened in binary mode.

mode – The mode. If given, this argument must be “r”.

formats – A list or tuple of formats to attempt to load the file in. This can be used to restrict the set of formats checked. Pass None to try all supported formats. You can print the set of available formats by running python3 -m PIL or using the PIL.features.pilinfo() function.

RETURNS:
An Image object.

RAISES:
FileNotFoundError – If the file cannot be found.

PIL.UnidentifiedImageError – If the image cannot be opened and identified.

ValueError – If the mode is not “r”, or if a StringIO instance is used for fp.

TypeError – If formats is not None, a list or a tuple.

Warning

To protect against potential DOS attacks caused by “decompression bombs” (i.e. malicious files which decompress into a huge amount of data and are designed to crash or cause disruption by using up a lot of memory), Pillow will issue a DecompressionBombWarning if the number of pixels in an image is over a certain limit, MAX_IMAGE_PIXELS.

This threshold can be changed by setting MAX_IMAGE_PIXELS. It can be disabled by setting Image.MAX_IMAGE_PIXELS = None.

If desired, the warning can be turned into an error with warnings.simplefilter('error', Image.DecompressionBombWarning) or suppressed entirely with warnings.simplefilter('ignore', Image.DecompressionBombWarning). See also the logging documentation to have warnings output to the logging facility instead of stderr.

If the number of pixels is greater than twice MAX_IMAGE_PIXELS, then a DecompressionBombError will be raised instead.

Image processing
PIL.Image.alpha_composite(im1, im2)[source]
Alpha composite im2 over im1.

PARAMETERS:
im1 – The first image. Must have mode RGBA.

im2 – The second image. Must have mode RGBA, and the same size as the first image.

RETURNS:
An Image object.

PIL.Image.blend(im1, im2, alpha)[source]
Creates a new image by interpolating between two input images, using a constant alpha:

out = image1 * (1.0 - alpha) + image2 * alpha
PARAMETERS:
im1 – The first image.

im2 – The second image. Must have the same mode and size as the first image.

alpha – The interpolation alpha factor. If alpha is 0.0, a copy of the first image is returned. If alpha is 1.0, a copy of the second image is returned. There are no restrictions on the alpha value. If necessary, the result is clipped to fit into the allowed output range.

RETURNS:
An Image object.

PIL.Image.composite(image1, image2, mask)[source]
Create composite image by blending images using a transparency mask.

PARAMETERS:
image1 – The first image.

image2 – The second image. Must have the same mode and size as the first image.

mask – A mask image. This image can have mode “1”, “L”, or “RGBA”, and must have the same size as the other two images.

PIL.Image.eval(image, *args)[source]
Applies the function (which should take one argument) to each pixel in the given image. If the image has more than one band, the same function is applied to each band. Note that the function is evaluated once for each possible pixel value, so you cannot use random components or other generators.

PARAMETERS:
image – The input image.

function – A function object, taking one integer argument.

RETURNS:
An Image object.

PIL.Image.merge(mode, bands)[source]
Merge a set of single band images into a new multiband image.

PARAMETERS:
mode – The mode to use for the output image. See: Modes.

bands – A sequence containing one single-band image for each band in the output image. All bands must have the same size.

RETURNS:
An Image object.

Constructing images
PIL.Image.new(mode, size, color=0)[source]
Creates a new image with the given mode and size.

PARAMETERS:
mode – The mode to use for the new image. See: Modes.

size – A 2-tuple, containing (width, height) in pixels.

color – What color to use for the image. Default is black. If given, this should be a single integer or floating point value for single-band modes, and a tuple for multi-band modes (one value per band). When creating RGB images, you can also use color strings as supported by the ImageColor module. If the color is None, the image is not initialised.

RETURNS:
An Image object.

PIL.Image.fromarray(obj, mode=None)[source]
Creates an image memory from an object exporting the array interface (using the buffer protocol).

If obj is not contiguous, then the tobytes method is called and frombuffer() is used.

If you have an image in NumPy:

from PIL import Image
import numpy as np
im = Image.open("hopper.jpg")
a = np.asarray(im)
Then this can be used to convert it to a Pillow image:

im = Image.fromarray(a)
PARAMETERS:
obj – Object with array interface

mode –

Optional mode to use when reading obj. Will be determined from type if None.

This will not be used to convert the data after reading, but will be used to change how the data is read:

from PIL import Image
import numpy as np
a = np.full((1, 1), 300)
im = Image.fromarray(a, mode="L")
im.getpixel((0, 0))  # 44
im = Image.fromarray(a, mode="RGB")
im.getpixel((0, 0))  # (44, 1, 0)
See: Modes for general information about modes.

RETURNS:
An image object.

New in version 1.1.6.

PIL.Image.frombytes(mode, size, data, decoder_name='raw', *args)[source]
Creates a copy of an image memory from pixel data in a buffer.

In its simplest form, this function takes three arguments (mode, size, and unpacked pixel data).

You can also use any pixel decoder supported by PIL. For more information on available decoders, see the section Writing Your Own File Codec.

Note that this function decodes pixel data only, not entire images. If you have an entire image in a string, wrap it in a BytesIO object, and use open() to load it.

PARAMETERS:
mode – The image mode. See: Modes.

size – The image size.

data – A byte buffer containing raw data for the given mode.

decoder_name – What decoder to use.

args – Additional parameters for the given decoder.

RETURNS:
An Image object.

PIL.Image.frombuffer(mode, size, data, decoder_name='raw', *args)[source]
Creates an image memory referencing pixel data in a byte buffer.

This function is similar to frombytes(), but uses data in the byte buffer, where possible. This means that changes to the original buffer object are reflected in this image). Not all modes can share memory; supported modes include “L”, “RGBX”, “RGBA”, and “CMYK”.

Note that this function decodes pixel data only, not entire images. If you have an entire image file in a string, wrap it in a BytesIO object, and use open() to load it.

In the current version, the default parameters used for the “raw” decoder differs from that used for frombytes(). This is a bug, and will probably be fixed in a future release. The current release issues a warning if you do this; to disable the warning, you should provide the full set of parameters. See below for details.

PARAMETERS:
mode – The image mode. See: Modes.

size – The image size.

data – A bytes or other buffer object containing raw data for the given mode.

decoder_name – What decoder to use.

args –

Additional parameters for the given decoder. For the default encoder (“raw”), it’s recommended that you provide the full set of parameters:

frombuffer(mode, size, data, "raw", mode, 0, 1)
RETURNS:
An Image object.

New in version 1.1.4.

Generating images
PIL.Image.effect_mandelbrot(size, extent, quality)[source]
Generate a Mandelbrot set covering the given extent.

PARAMETERS:
size – The requested size in pixels, as a 2-tuple: (width, height).

extent – The extent to cover, as a 4-tuple: (x0, y0, x1, y1).

quality – Quality.

PIL.Image.effect_noise(size, sigma)[source]
Generate Gaussian noise centered around 128.

PARAMETERS:
size – The requested size in pixels, as a 2-tuple: (width, height).

sigma – Standard deviation of noise.

PIL.Image.linear_gradient(mode)[source]
Generate 256x256 linear gradient from black to white, top to bottom.

PARAMETERS:
mode – Input mode.

PIL.Image.radial_gradient(mode)[source]
Generate 256x256 radial gradient from black to white, centre to edge.

PARAMETERS:
mode – Input mode.

Registering plugins
Note

These functions are for use by plugin authors. Application authors can ignore them.

PIL.Image.register_open(id, factory, accept=None)[source]
Register an image file plugin. This function should not be used in application code.

PARAMETERS:
id – An image format identifier.

factory – An image file factory method.

accept – An optional function that can be used to quickly reject images having another format.

PIL.Image.register_mime(id, mimetype)[source]
Registers an image MIME type. This function should not be used in application code.

PARAMETERS:
id – An image format identifier.

mimetype – The image MIME type for this format.

PIL.Image.register_save(id, driver)[source]
Registers an image save function. This function should not be used in application code.

PARAMETERS:
id – An image format identifier.

driver – A function to save images in this format.

PIL.Image.register_save_all(id, driver)[source]
Registers an image function to save all the frames of a multiframe format. This function should not be used in application code.

PARAMETERS:
id – An image format identifier.

driver – A function to save images in this format.

PIL.Image.register_extension(id, extension)[source]
Registers an image extension. This function should not be used in application code.

PARAMETERS:
id – An image format identifier.

extension – An extension used for this format.

PIL.Image.register_extensions(id, extensions)[source]
Registers image extensions. This function should not be used in application code.

PARAMETERS:
id – An image format identifier.

extensions – A list of extensions used for this format.

PIL.Image.registered_extensions()[source]
Returns a dictionary containing all file extensions belonging to registered plugins

PIL.Image.register_decoder(name, decoder)[source]
Registers an image decoder. This function should not be used in application code.

PARAMETERS:
name – The name of the decoder

decoder – A callable(mode, args) that returns an ImageFile.PyDecoder object

New in version 4.1.0.

PIL.Image.register_encoder(name, encoder)[source]
Registers an image encoder. This function should not be used in application code.

PARAMETERS:
name – The name of the encoder

encoder – A callable(mode, args) that returns an ImageFile.PyEncoder object

New in version 4.1.0.

The Image Class
class PIL.Image.Image[source]
This class represents an image object. To create Image objects, use the appropriate factory functions. There’s hardly ever any reason to call the Image constructor directly.

open()

new()

frombytes()

An instance of the Image class has the following methods. Unless otherwise stated, all methods return a new instance of the Image class, holding the resulting image.

Image.alpha_composite(im, dest=(0, 0), source=(0, 0))[source]
‘In-place’ analog of Image.alpha_composite. Composites an image onto this image.

PARAMETERS:
im – image to composite over this one

dest – Optional 2 tuple (left, top) specifying the upper left corner in this (destination) image.

source – Optional 2 (left, top) tuple for the upper left corner in the overlay source image, or 4 tuple (left, top, right, bottom) for the bounds of the source rectangle

Performance Note: Not currently implemented in-place in the core layer.

Image.apply_transparency()[source]
If a P mode image has a “transparency” key in the info dictionary, remove the key and instead apply the transparency to the palette. Otherwise, the image is unchanged.

Image.convert(mode=None, matrix=None, dither=None, palette=Palette.WEB, colors=256)[source]
Returns a converted copy of this image. For the “P” mode, this method translates pixels through the palette. If mode is omitted, a mode is chosen so that all information in the image and the palette can be represented without a palette.

The current version supports all possible conversions between “L”, “RGB” and “CMYK”. The matrix argument only supports “L” and “RGB”.

When translating a color image to greyscale (mode “L”), the library uses the ITU-R 601-2 luma transform:

L = R * 299/1000 + G * 587/1000 + B * 114/1000
The default method of converting a greyscale (“L”) or “RGB” image into a bilevel (mode “1”) image uses Floyd-Steinberg dither to approximate the original image luminosity levels. If dither is None, all values larger than 127 are set to 255 (white), all other values to 0 (black). To use other thresholds, use the point() method.

When converting from “RGBA” to “P” without a matrix argument, this passes the operation to quantize(), and dither and palette are ignored.

When converting from “PA”, if an “RGBA” palette is present, the alpha channel from the image will be used instead of the values from the palette.

PARAMETERS:
mode – The requested mode. See: Modes.

matrix – An optional conversion matrix. If given, this should be 4- or 12-tuple containing floating point values.

dither – Dithering method, used when converting from mode “RGB” to “P” or from “RGB” or “L” to “1”. Available methods are Dither.NONE or Dither.FLOYDSTEINBERG (default). Note that this is not used when matrix is supplied.

palette – Palette to use when converting from mode “RGB” to “P”. Available palettes are Palette.WEB or Palette.ADAPTIVE.

colors – Number of colors to use for the Palette.ADAPTIVE palette. Defaults to 256.

RETURN TYPE:
Image

RETURNS:
An Image object.

The following example converts an RGB image (linearly calibrated according to ITU-R 709, using the D65 luminant) to the CIE XYZ color space:

rgb2xyz = (
    0.412453, 0.357580, 0.180423, 0,
    0.212671, 0.715160, 0.072169, 0,
    0.019334, 0.119193, 0.950227, 0)
out = im.convert("RGB", rgb2xyz)
Image.copy()[source]
Copies this image. Use this method if you wish to paste things into an image, but still retain the original.

RETURN TYPE:
Image

RETURNS:
An Image object.

Image.crop(box=None)[source]
Returns a rectangular region from this image. The box is a 4-tuple defining the left, upper, right, and lower pixel coordinate. See Coordinate System.

Note: Prior to Pillow 3.4.0, this was a lazy operation.

PARAMETERS:
box – The crop rectangle, as a (left, upper, right, lower)-tuple.

RETURN TYPE:
Image

RETURNS:
An Image object.

This crops the input image with the provided coordinates:

from PIL import Image

with Image.open("hopper.jpg") as im:

    # The crop method from the Image module takes four coordinates as input.
    # The right can also be represented as (left+width)
    # and lower can be represented as (upper+height).
    (left, upper, right, lower) = (20, 20, 100, 100)

    # Here the image "im" is cropped and assigned to new variable im_crop
    im_crop = im.crop((left, upper, right, lower))
Image.draft(mode, size)[source]
Configures the image file loader so it returns a version of the image that as closely as possible matches the given mode and size. For example, you can use this method to convert a color JPEG to greyscale while loading it.

If any changes are made, returns a tuple with the chosen mode and box with coordinates of the original image within the altered one.

Note that this method modifies the Image object in place. If the image has already been loaded, this method has no effect.

Note: This method is not implemented for most images. It is currently implemented only for JPEG and MPO images.

PARAMETERS:
mode – The requested mode.

size – The requested size in pixels, as a 2-tuple: (width, height).

Image.effect_spread(distance)[source]
Randomly spread pixels in an image.

PARAMETERS:
distance – Distance to spread pixels.

Image.entropy(mask=None, extrema=None)[source]
Calculates and returns the entropy for the image.

A bilevel image (mode “1”) is treated as a greyscale (“L”) image by this method.

If a mask is provided, the method employs the histogram for those parts of the image where the mask image is non-zero. The mask image must have the same size as the image, and be either a bi-level image (mode “1”) or a greyscale image (“L”).

PARAMETERS:
mask – An optional mask.

extrema – An optional tuple of manually-specified extrema.

RETURNS:
A float value representing the image entropy

Image.filter(filter)[source]
Filters this image using the given filter. For a list of available filters, see the ImageFilter module.

PARAMETERS:
filter – Filter kernel.

RETURNS:
An Image object.

This blurs the input image using a filter from the ImageFilter module:

from PIL import Image, ImageFilter

with Image.open("hopper.jpg") as im:

    # Blur the input image using the filter ImageFilter.BLUR
    im_blurred = im.filter(filter=ImageFilter.BLUR)
Image.frombytes(data, decoder_name='raw', *args)[source]
Loads this image with pixel data from a bytes object.

This method is similar to the frombytes() function, but loads data into this image instead of creating a new image object.

Image.getbands()[source]
Returns a tuple containing the name of each band in this image. For example, getbands on an RGB image returns (“R”, “G”, “B”).

RETURNS:
A tuple containing band names.

RETURN TYPE:
tuple

This helps to get the bands of the input image:

from PIL import Image

with Image.open("hopper.jpg") as im:
    print(im.getbands())  # Returns ('R', 'G', 'B')
Image.getbbox()[source]
Calculates the bounding box of the non-zero regions in the image.

RETURNS:
The bounding box is returned as a 4-tuple defining the left, upper, right, and lower pixel coordinate. See Coordinate System. If the image is completely empty, this method returns None.

This helps to get the bounding box coordinates of the input image:

from PIL import Image

with Image.open("hopper.jpg") as im:
    print(im.getbbox())
    # Returns four coordinates in the format (left, upper, right, lower)
Image.getchannel(channel)[source]
Returns an image containing a single channel of the source image.

PARAMETERS:
channel – What channel to return. Could be index (0 for “R” channel of “RGB”) or channel name (“A” for alpha channel of “RGBA”).

RETURNS:
An image in “L” mode.

New in version 4.3.0.

Image.getcolors(maxcolors=256)[source]
Returns a list of colors used in this image.

The colors will be in the image’s mode. For example, an RGB image will return a tuple of (red, green, blue) color values, and a P image will return the index of the color in the palette.

PARAMETERS:
maxcolors – Maximum number of colors. If this number is exceeded, this method returns None. The default limit is 256 colors.

RETURNS:
An unsorted list of (count, pixel) values.

Image.getdata(band=None)[source]
Returns the contents of this image as a sequence object containing pixel values. The sequence object is flattened, so that values for line one follow directly after the values of line zero, and so on.

Note that the sequence object returned by this method is an internal PIL data type, which only supports certain sequence operations. To convert it to an ordinary sequence (e.g. for printing), use list(im.getdata()).

PARAMETERS:
band – What band to return. The default is to return all bands. To return a single band, pass in the index value (e.g. 0 to get the “R” band from an “RGB” image).

RETURNS:
A sequence-like object.

Image.getexif()[source]
Image.getextrema()[source]
Gets the minimum and maximum pixel values for each band in the image.

RETURNS:
For a single-band image, a 2-tuple containing the minimum and maximum pixel value. For a multi-band image, a tuple containing one 2-tuple for each band.

Image.getpalette(rawmode='RGB')[source]
Returns the image palette as a list.

PARAMETERS:
rawmode –

The mode in which to return the palette. None will return the palette in its current mode.

New in version 9.1.0.

RETURNS:
A list of color values [r, g, b, …], or None if the image has no palette.

Image.getpixel(xy)[source]
Returns the pixel value at a given position.

PARAMETERS:
xy – The coordinate, given as (x, y). See Coordinate System.

RETURNS:
The pixel value. If the image is a multi-layer image, this method returns a tuple.

Image.getprojection()[source]
Get projection to x and y axes

RETURNS:
Two sequences, indicating where there are non-zero pixels along the X-axis and the Y-axis, respectively.

Image.histogram(mask=None, extrema=None)[source]
Returns a histogram for the image. The histogram is returned as a list of pixel counts, one for each pixel value in the source image. Counts are grouped into 256 bins for each band, even if the image has more than 8 bits per band. If the image has more than one band, the histograms for all bands are concatenated (for example, the histogram for an “RGB” image contains 768 values).

A bilevel image (mode “1”) is treated as a greyscale (“L”) image by this method.

If a mask is provided, the method returns a histogram for those parts of the image where the mask image is non-zero. The mask image must have the same size as the image, and be either a bi-level image (mode “1”) or a greyscale image (“L”).

PARAMETERS:
mask – An optional mask.

extrema – An optional tuple of manually-specified extrema.

RETURNS:
A list containing pixel counts.

Image.paste(im, box=None, mask=None)[source]
Pastes another image into this image. The box argument is either a 2-tuple giving the upper left corner, a 4-tuple defining the left, upper, right, and lower pixel coordinate, or None (same as (0, 0)). See Coordinate System. If a 4-tuple is given, the size of the pasted image must match the size of the region.

If the modes don’t match, the pasted image is converted to the mode of this image (see the convert() method for details).

Instead of an image, the source can be a integer or tuple containing pixel values. The method then fills the region with the given color. When creating RGB images, you can also use color strings as supported by the ImageColor module.

If a mask is given, this method updates only the regions indicated by the mask. You can use either “1”, “L”, “LA”, “RGBA” or “RGBa” images (if present, the alpha band is used as mask). Where the mask is 255, the given image is copied as is. Where the mask is 0, the current value is preserved. Intermediate values will mix the two images together, including their alpha channels if they have them.

See alpha_composite() if you want to combine images with respect to their alpha channels.

PARAMETERS:
im – Source image or pixel value (integer or tuple).

box –

An optional 4-tuple giving the region to paste into. If a 2-tuple is used instead, it’s treated as the upper left corner. If omitted or None, the source is pasted into the upper left corner.

If an image is given as the second argument and there is no third, the box defaults to (0, 0), and the second argument is interpreted as a mask image.

mask – An optional mask image.

Image.point(lut, mode=None)[source]
Maps this image through a lookup table or function.

PARAMETERS:
lut –

A lookup table, containing 256 (or 65536 if self.mode==”I” and mode == “L”) values per band in the image. A function can be used instead, it should take a single argument. The function is called once for each possible pixel value, and the resulting table is applied to all bands of the image.

It may also be an ImagePointHandler object:

class Example(Image.ImagePointHandler):
  def point(self, data):
    # Return result
mode – Output mode (default is same as input). In the current version, this can only be used if the source image has mode “L” or “P”, and the output has mode “1” or the source image mode is “I” and the output mode is “L”.

RETURNS:
An Image object.

Image.putalpha(alpha)[source]
Adds or replaces the alpha layer in this image. If the image does not have an alpha layer, it’s converted to “LA” or “RGBA”. The new layer must be either “L” or “1”.

PARAMETERS:
alpha – The new alpha layer. This can either be an “L” or “1” image having the same size as this image, or an integer or other color value.

Image.putdata(data, scale=1.0, offset=0.0)[source]
Copies pixel data from a flattened sequence object into the image. The values should start at the upper left corner (0, 0), continue to the end of the line, followed directly by the first value of the second line, and so on. Data will be read until either the image or the sequence ends. The scale and offset values are used to adjust the sequence values: pixel = value*scale + offset.

PARAMETERS:
data – A flattened sequence object.

scale – An optional scale value. The default is 1.0.

offset – An optional offset value. The default is 0.0.

Image.putpalette(data, rawmode='RGB')[source]
Attaches a palette to this image. The image must be a “P”, “PA”, “L” or “LA” image.

The palette sequence must contain at most 256 colors, made up of one integer value for each channel in the raw mode. For example, if the raw mode is “RGB”, then it can contain at most 768 values, made up of red, green and blue values for the corresponding pixel index in the 256 colors. If the raw mode is “RGBA”, then it can contain at most 1024 values, containing red, green, blue and alpha values.

Alternatively, an 8-bit string may be used instead of an integer sequence.

PARAMETERS:
data – A palette sequence (either a list or a string).

rawmode – The raw mode of the palette. Either “RGB”, “RGBA”, or a mode that can be transformed to “RGB” or “RGBA” (e.g. “R”, “BGR;15”, “RGBA;L”).

Image.putpixel(xy, value)[source]
Modifies the pixel at the given position. The color is given as a single numerical value for single-band images, and a tuple for multi-band images. In addition to this, RGB and RGBA tuples are accepted for P and PA images.

Note that this method is relatively slow. For more extensive changes, use paste() or the ImageDraw module instead.

See:

paste()

putdata()

ImageDraw

PARAMETERS:
xy – The pixel coordinate, given as (x, y). See Coordinate System.

value – The pixel value.

Image.quantize(colors=256, method=None, kmeans=0, palette=None, dither=Dither.FLOYDSTEINBERG)[source]
Convert the image to ‘P’ mode with the specified number of colors.

PARAMETERS:
colors – The desired number of colors, <= 256

method –

Quantize.MEDIANCUT (median cut), Quantize.MAXCOVERAGE (maximum coverage), Quantize.FASTOCTREE (fast octree), Quantize.LIBIMAGEQUANT (libimagequant; check support using PIL.features.check_feature() with feature="libimagequant").

By default, Quantize.MEDIANCUT will be used.

The exception to this is RGBA images. Quantize.MEDIANCUT and Quantize.MAXCOVERAGE do not support RGBA images, so Quantize.FASTOCTREE is used by default instead.

kmeans – Integer

palette – Quantize to the palette of given PIL.Image.Image.

dither – Dithering method, used when converting from mode “RGB” to “P” or from “RGB” or “L” to “1”. Available methods are Dither.NONE or Dither.FLOYDSTEINBERG (default).

RETURNS:
A new image

Image.reduce(factor, box=None)[source]
Returns a copy of the image reduced factor times. If the size of the image is not dividable by factor, the resulting size will be rounded up.

PARAMETERS:
factor – A greater than 0 integer or tuple of two integers for width and height separately.

box – An optional 4-tuple of ints providing the source image region to be reduced. The values must be within (0, 0, width, height) rectangle. If omitted or None, the entire source is used.

Image.remap_palette(dest_map, source_palette=None)[source]
Rewrites the image to reorder the palette.

PARAMETERS:
dest_map – A list of indexes into the original palette. e.g. [1,0] would swap a two item palette, and list(range(256)) is the identity transform.

source_palette – Bytes or None.

RETURNS:
An Image object.

Image.resize(size, resample=None, box=None, reducing_gap=None)[source]
Returns a resized copy of this image.

PARAMETERS:
size – The requested size in pixels, as a 2-tuple: (width, height).

resample – An optional resampling filter. This can be one of Resampling.NEAREST, Resampling.BOX, Resampling.BILINEAR, Resampling.HAMMING, Resampling.BICUBIC or Resampling.LANCZOS. If the image has mode “1” or “P”, it is always set to Resampling.NEAREST. If the image mode specifies a number of bits, such as “I;16”, then the default filter is Resampling.NEAREST. Otherwise, the default filter is Resampling.BICUBIC. See: Filters.

box – An optional 4-tuple of floats providing the source image region to be scaled. The values must be within (0, 0, width, height) rectangle. If omitted or None, the entire source is used.

reducing_gap – Apply optimization by resizing the image in two steps. First, reducing the image by integer times using reduce(). Second, resizing using regular resampling. The last step changes size no less than by reducing_gap times. reducing_gap may be None (no first step is performed) or should be greater than 1.0. The bigger reducing_gap, the closer the result to the fair resampling. The smaller reducing_gap, the faster resizing. With reducing_gap greater or equal to 3.0, the result is indistinguishable from fair resampling in most cases. The default value is None (no optimization).

RETURNS:
An Image object.

This resizes the given image from (width, height) to (width/2, height/2):

from PIL import Image

with Image.open("hopper.jpg") as im:

    # Provide the target width and height of the image
    (width, height) = (im.width // 2, im.height // 2)
    im_resized = im.resize((width, height))
Image.rotate(angle, resample=Resampling.NEAREST, expand=0, center=None, translate=None, fillcolor=None)[source]
Returns a rotated copy of this image. This method returns a copy of this image, rotated the given number of degrees counter clockwise around its centre.

PARAMETERS:
angle – In degrees counter clockwise.

resample – An optional resampling filter. This can be one of Resampling.NEAREST (use nearest neighbour), Resampling.BILINEAR (linear interpolation in a 2x2 environment), or Resampling.BICUBIC (cubic spline interpolation in a 4x4 environment). If omitted, or if the image has mode “1” or “P”, it is set to Resampling.NEAREST. See Filters.

expand – Optional expansion flag. If true, expands the output image to make it large enough to hold the entire rotated image. If false or omitted, make the output image the same size as the input image. Note that the expand flag assumes rotation around the center and no translation.

center – Optional center of rotation (a 2-tuple). Origin is the upper left corner. Default is the center of the image.

translate – An optional post-rotate translation (a 2-tuple).

fillcolor – An optional color for area outside the rotated image.

RETURNS:
An Image object.

This rotates the input image by theta degrees counter clockwise:

from PIL import Image

with Image.open("hopper.jpg") as im:

    # Rotate the image by 60 degrees counter clockwise
    theta = 60
    # Angle is in degrees counter clockwise
    im_rotated = im.rotate(angle=theta)
Image.save(fp, format=None, **params)[source]
Saves this image under the given filename. If no format is specified, the format to use is determined from the filename extension, if possible.

Keyword options can be used to provide additional instructions to the writer. If a writer doesn’t recognise an option, it is silently ignored. The available options are described in the image format documentation for each writer.

You can use a file object instead of a filename. In this case, you must always specify the format. The file object must implement the seek, tell, and write methods, and be opened in binary mode.

PARAMETERS:
fp – A filename (string), pathlib.Path object or file object.

format – Optional format override. If omitted, the format to use is determined from the filename extension. If a file object was used instead of a filename, this parameter should always be used.

params – Extra parameters to the image writer.

RETURNS:
None

RAISES:
ValueError – If the output format could not be determined from the file name. Use the format option to solve this.

OSError – If the file could not be written. The file may have been created, and may contain partial data.

Image.seek(frame)[source]
Seeks to the given frame in this sequence file. If you seek beyond the end of the sequence, the method raises an EOFError exception. When a sequence file is opened, the library automatically seeks to frame 0.

See tell().

If defined, n_frames refers to the number of available frames.

PARAMETERS:
frame – Frame number, starting at 0.

RAISES:
EOFError – If the call attempts to seek beyond the end of the sequence.

Image.show(title=None)[source]
Displays this image. This method is mainly intended for debugging purposes.

This method calls PIL.ImageShow.show() internally. You can use PIL.ImageShow.register() to override its default behaviour.

The image is first saved to a temporary file. By default, it will be in PNG format.

On Unix, the image is then opened using the display, eog or xv utility, depending on which one can be found.

On macOS, the image is opened with the native Preview application.

On Windows, the image is opened with the standard PNG display utility.

PARAMETERS:
title – Optional title to use for the image window, where possible.

Image.split()[source]
Split this image into individual bands. This method returns a tuple of individual image bands from an image. For example, splitting an “RGB” image creates three new images each containing a copy of one of the original bands (red, green, blue).

If you need only one band, getchannel() method can be more convenient and faster.

RETURNS:
A tuple containing bands.

Image.tell()[source]
Returns the current frame number. See seek().

If defined, n_frames refers to the number of available frames.

RETURNS:
Frame number, starting with 0.

Image.thumbnail(size, resample=Resampling.BICUBIC, reducing_gap=2.0)[source]
Make this image into a thumbnail. This method modifies the image to contain a thumbnail version of itself, no larger than the given size. This method calculates an appropriate thumbnail size to preserve the aspect of the image, calls the draft() method to configure the file reader (where applicable), and finally resizes the image.

Note that this function modifies the Image object in place. If you need to use the full resolution image as well, apply this method to a copy() of the original image.

PARAMETERS:
size – The requested size in pixels, as a 2-tuple: (width, height).

resample – Optional resampling filter. This can be one of Resampling.NEAREST, Resampling.BOX, Resampling.BILINEAR, Resampling.HAMMING, Resampling.BICUBIC or Resampling.LANCZOS. If omitted, it defaults to Resampling.BICUBIC. (was Resampling.NEAREST prior to version 2.5.0). See: Filters.

reducing_gap – Apply optimization by resizing the image in two steps. First, reducing the image by integer times using reduce() or draft() for JPEG images. Second, resizing using regular resampling. The last step changes size no less than by reducing_gap times. reducing_gap may be None (no first step is performed) or should be greater than 1.0. The bigger reducing_gap, the closer the result to the fair resampling. The smaller reducing_gap, the faster resizing. With reducing_gap greater or equal to 3.0, the result is indistinguishable from fair resampling in most cases. The default value is 2.0 (very close to fair resampling while still being faster in many cases).

RETURNS:
None

Image.tobitmap(name='image')[source]
Returns the image converted to an X11 bitmap.

Note

This method only works for mode “1” images.

PARAMETERS:
name – The name prefix to use for the bitmap variables.

RETURNS:
A string containing an X11 bitmap.

RAISES:
ValueError – If the mode is not “1”

Image.tobytes(encoder_name='raw', *args)[source]
Return image as a bytes object.

Warning

This method returns the raw image data from the internal storage. For compressed image data (e.g. PNG, JPEG) use save(), with a BytesIO parameter for in-memory data.

PARAMETERS:
encoder_name –

What encoder to use. The default is to use the standard “raw” encoder.

A list of C encoders can be seen under codecs section of the function array in _imaging.c. Python encoders are registered within the relevant plugins.

args – Extra arguments to the encoder.

RETURNS:
A bytes object.

Image.transform(size, method, data=None, resample=Resampling.NEAREST, fill=1, fillcolor=None)[source]
Transforms this image. This method creates a new image with the given size, and the same mode as the original, and copies data to the new image using the given transform.

PARAMETERS:
size – The output size in pixels, as a 2-tuple: (width, height).

method –

The transformation method. This is one of Transform.EXTENT (cut out a rectangular subregion), Transform.AFFINE (affine transform), Transform.PERSPECTIVE (perspective transform), Transform.QUAD (map a quadrilateral to a rectangle), or Transform.MESH (map a number of source quadrilaterals in one operation).

It may also be an ImageTransformHandler object:

class Example(Image.ImageTransformHandler):
    def transform(self, size, data, resample, fill=1):
        # Return result
It may also be an object with a method.getdata method that returns a tuple supplying new method and data values:

class Example:
    def getdata(self):
        method = Image.Transform.EXTENT
        data = (0, 0, 100, 100)
        return method, data
data – Extra data to the transformation method.

resample – Optional resampling filter. It can be one of Resampling.NEAREST (use nearest neighbour), Resampling.BILINEAR (linear interpolation in a 2x2 environment), or Resampling.BICUBIC (cubic spline interpolation in a 4x4 environment). If omitted, or if the image has mode “1” or “P”, it is set to Resampling.NEAREST. See: Filters.

fill – If method is an ImageTransformHandler object, this is one of the arguments passed to it. Otherwise, it is unused.

fillcolor – Optional fill color for the area outside the transform in the output image.

RETURNS:
An Image object.

Image.transpose(method)[source]
Transpose image (flip or rotate in 90 degree steps)

PARAMETERS:
method – One of Transpose.FLIP_LEFT_RIGHT, Transpose.FLIP_TOP_BOTTOM, Transpose.ROTATE_90, Transpose.ROTATE_180, Transpose.ROTATE_270, Transpose.TRANSPOSE or Transpose.TRANSVERSE.

RETURNS:
Returns a flipped or rotated copy of this image.

This flips the input image by using the Transpose.FLIP_LEFT_RIGHT method.

from PIL import Image

with Image.open("hopper.jpg") as im:

    # Flip the image from left to right
    im_flipped = im.transpose(method=Image.Transpose.FLIP_LEFT_RIGHT)
    # To flip the image from top to bottom,
    # use the method "Image.Transpose.FLIP_TOP_BOTTOM"
Image.verify()[source]
Verifies the contents of a file. For data read from a file, this method attempts to determine if the file is broken, without actually decoding the image data. If this method finds any problems, it raises suitable exceptions. If you need to load the image after using this method, you must reopen the image file.

Image.load()[source]
Allocates storage for the image and loads the pixel data. In normal cases, you don’t need to call this method, since the Image class automatically loads an opened image when it is accessed for the first time.

If the file associated with the image was opened by Pillow, then this method will close it. The exception to this is if the image has multiple frames, in which case the file will be left open for seek operations. See File Handling in Pillow for more information.

RETURNS:
An image access object.

RETURN TYPE:
PixelAccess Class or PIL.PyAccess

Image.close()[source]
Closes the file pointer, if possible.

This operation will destroy the image core and release its memory. The image data will be unusable afterward.

This function is required to close images that have multiple frames or have not had their file read and closed by the load() method. See File Handling in Pillow for more information.

Image Attributes
Instances of the Image class have the following attributes:

Image.filename: str
The filename or path of the source file. Only images created with the factory function open have a filename attribute. If the input is a file like object, the filename attribute is set to an empty string.

Image.format: Optional[str]
The file format of the source file. For images created by the library itself (via a factory function, or by running a method on an existing image), this attribute is set to None.

Image.mode: str
Image mode. This is a string specifying the pixel format used by the image. Typical values are “1”, “L”, “RGB”, or “CMYK.” See Modes for a full list.

Image.size: tuple[int]
Image size, in pixels. The size is given as a 2-tuple (width, height).

Image.width: int
Image width, in pixels.

Image.height: int
Image height, in pixels.

Image.palette: Optional[PIL.ImagePalette.ImagePalette]
Colour palette table, if any. If mode is “P” or “PA”, this should be an instance of the ImagePalette class. Otherwise, it should be set to None.

Image.info: dict
A dictionary holding data associated with the image. This dictionary is used by file handlers to pass on various non-image information read from the file. See documentation for the various file handlers for details.

Most methods ignore the dictionary when returning new images; since the keys are not standardized, it’s not possible for a method to know if the operation affects the dictionary. If you need the information later on, keep a reference to the info dictionary returned from the open method.

Unless noted elsewhere, this dictionary does not affect saving files.

Image.is_animated: bool
True if this image has more than one frame, or False otherwise.

This attribute is only defined by image plugins that support animated images. Plugins may leave this attribute undefined if they don’t support loading animated images, even if the given format supports animated images.

Given that this attribute is not present for all images use getattr(image, "is_animated", False) to check if Pillow is aware of multiple frames in an image regardless of its format.

See also

n_frames, seek() and tell()

Image.n_frames: int
The number of frames in this image.

This attribute is only defined by image plugins that support animated images. Plugins may leave this attribute undefined if they don’t support loading animated images, even if the given format supports animated images.

Given that this attribute is not present for all images use getattr(image, "n_frames", 1) to check the number of frames that Pillow is aware of in an image regardless of its format.

See also

is_animated, seek() and tell()

Classes
class PIL.Image.Exif[source]
Bases: MutableMapping

bigtiff = False
endian = None
get_ifd(tag)[source]
hide_offsets()[source]
load(data)[source]
load_from_fp(fp, offset=None)[source]
tobytes(offset=8)[source]
class PIL.Image.ImagePointHandler[source]
Used as a mixin by point transforms (for use with point())

class PIL.Image.ImageTransformHandler[source]
Used as a mixin by geometry transforms (for use with transform())

Constants
PIL.Image.NONE
PIL.Image.MAX_IMAGE_PIXELS
Set to 89,478,485, approximately 0.25GB for a 24-bit (3 bpp) image. See open() for more information about how this is used.

Transpose methods
Used to specify the Image.transpose() method to use.

class PIL.Image.Transpose(value)[source]
An enumeration.

FLIP_LEFT_RIGHT = 0
FLIP_TOP_BOTTOM = 1
ROTATE_180 = 3
ROTATE_270 = 4
ROTATE_90 = 2
TRANSPOSE = 5
TRANSVERSE = 6
Transform methods
Used to specify the Image.transform() method to use.

class PIL.Image.Transform[source]
AFFINE
Affine transform

EXTENT
Cut out a rectangular subregion

PERSPECTIVE
Perspective transform

QUAD
Map a quadrilateral to a rectangle

MESH
Map a number of source quadrilaterals in one operation

Resampling filters
See Filters for details.

class PIL.Image.Resampling(value)[source]
An enumeration.

BICUBIC = 3
BILINEAR = 2
BOX = 4
HAMMING = 5
LANCZOS = 1
NEAREST = 0
Some deprecated filters are also available under the following names:

PIL.Image.NONE = Resampling.NEAREST
PIL.Image.LINEAR = Resampling.BILINEAR
PIL.Image.CUBIC = Resampling.BICUBIC
PIL.Image.ANTIALIAS = Resampling.LANCZOS
Dither modes
Used to specify the dithering method to use for the convert() and quantize() methods.

class PIL.Image.Dither[source]
NONE
No dither

ORDERED
Not implemented

RASTERIZE
Not implemented

FLOYDSTEINBERG
Floyd-Steinberg dither

Palettes
Used to specify the pallete to use for the convert() method.

class PIL.Image.Palette(value)[source]
An enumeration.

ADAPTIVE = 1
WEB = 0
Quantization methods
Used to specify the quantization method to use for the quantize() method.

class PIL.Image.Quantize[source]
MEDIANCUT
Median cut. Default method, except for RGBA images. This method does not support RGBA images.

MAXCOVERAGE
Maximum coverage. This method does not support RGBA images.

FASTOCTREE
Fast octree. Default method for RGBA images.

LIBIMAGEQUANT
libimagequant

Check support using PIL.features.check_feature() with feature="libimagequant".



ImageChops (“Channel Operations”) Module
The ImageChops module contains a number of arithmetical image operations, called channel operations (“chops”). These can be used for various purposes, including special effects, image compositions, algorithmic painting, and more.

For more pre-made operations, see ImageOps.

At this time, most channel operations are only implemented for 8-bit images (e.g. “L” and “RGB”).

Functions
Most channel operations take one or two image arguments and returns a new image. Unless otherwise noted, the result of a channel operation is always clipped to the range 0 to MAX (which is 255 for all modes supported by the operations in this module).

PIL.ImageChops.add(image1, image2, scale=1.0, offset=0)[source]
Adds two images, dividing the result by scale and adding the offset. If omitted, scale defaults to 1.0, and offset to 0.0.

out = ((image1 + image2) / scale + offset)
RETURN TYPE:
Image

PIL.ImageChops.add_modulo(image1, image2)[source]
Add two images, without clipping the result.

out = ((image1 + image2) % MAX)
RETURN TYPE:
Image

PIL.ImageChops.blend(image1, image2, alpha)[source]
Blend images using constant transparency weight. Alias for PIL.Image.blend().

RETURN TYPE:
Image

PIL.ImageChops.composite(image1, image2, mask)[source]
Create composite using transparency mask. Alias for PIL.Image.composite().

RETURN TYPE:
Image

PIL.ImageChops.constant(image, value)[source]
Fill a channel with a given grey level.

RETURN TYPE:
Image

PIL.ImageChops.darker(image1, image2)[source]
Compares the two images, pixel by pixel, and returns a new image containing the darker values.

out = min(image1, image2)
RETURN TYPE:
Image

PIL.ImageChops.difference(image1, image2)[source]
Returns the absolute value of the pixel-by-pixel difference between the two images.

out = abs(image1 - image2)
RETURN TYPE:
Image

PIL.ImageChops.duplicate(image)[source]
Copy a channel. Alias for PIL.Image.Image.copy().

RETURN TYPE:
Image

PIL.ImageChops.invert(image)[source]
Invert an image (channel).

out = MAX - image
RETURN TYPE:
Image

PIL.ImageChops.lighter(image1, image2)[source]
Compares the two images, pixel by pixel, and returns a new image containing the lighter values.

out = max(image1, image2)
RETURN TYPE:
Image

PIL.ImageChops.logical_and(image1, image2)[source]
Logical AND between two images.

Both of the images must have mode “1”. If you would like to perform a logical AND on an image with a mode other than “1”, try multiply() instead, using a black-and-white mask as the second image.

out = ((image1 and image2) % MAX)
RETURN TYPE:
Image

PIL.ImageChops.logical_or(image1, image2)[source]
Logical OR between two images.

Both of the images must have mode “1”.

out = ((image1 or image2) % MAX)
RETURN TYPE:
Image

PIL.ImageChops.logical_xor(image1, image2)[source]
Logical XOR between two images.

Both of the images must have mode “1”.

out = ((bool(image1) != bool(image2)) % MAX)
RETURN TYPE:
Image

PIL.ImageChops.multiply(image1, image2)[source]
Superimposes two images on top of each other.

If you multiply an image with a solid black image, the result is black. If you multiply with a solid white image, the image is unaffected.

out = image1 * image2 / MAX
RETURN TYPE:
Image

PIL.ImageChops.soft_light(image1, image2)[source]
Superimposes two images on top of each other using the Soft Light algorithm

RETURN TYPE:
Image

PIL.ImageChops.hard_light(image1, image2)[source]
Superimposes two images on top of each other using the Hard Light algorithm

RETURN TYPE:
Image

PIL.ImageChops.overlay(image1, image2)[source]
Superimposes two images on top of each other using the Overlay algorithm

RETURN TYPE:
Image

PIL.ImageChops.offset(image, xoffset, yoffset=None)[source]
Returns a copy of the image where data has been offset by the given distances. Data wraps around the edges. If yoffset is omitted, it is assumed to be equal to xoffset.

PARAMETERS:
image – Input image.

xoffset – The horizontal distance.

yoffset – The vertical distance. If omitted, both distances are set to the same value.

RETURN TYPE:
Image

PIL.ImageChops.screen(image1, image2)[source]
Superimposes two inverted images on top of each other.

out = MAX - ((MAX - image1) * (MAX - image2) / MAX)
RETURN TYPE:
Image

PIL.ImageChops.subtract(image1, image2, scale=1.0, offset=0)[source]
Subtracts two images, dividing the result by scale and adding the offset. If omitted, scale defaults to 1.0, and offset to 0.0.

out = ((image1 - image2) / scale + offset)
RETURN TYPE:
Image

PIL.ImageChops.subtract_modulo(image1, image2)[source]
Subtract two images, without clipping the result.

out = ((image1 - image2) % MAX)
RETURN TYPE:
Image




ImageCms Module
The ImageCms module provides color profile management support using the LittleCMS2 color management engine, based on Kevin Cazabon’s PyCMS library.

class PIL.ImageCms.ImageCmsTransform(input, output, input_mode, output_mode, intent=Intent.PERCEPTUAL, proof=None, proof_intent=Intent.ABSOLUTE_COLORIMETRIC, flags=0)[source]
Transform. This can be used with the procedural API, or with the standard point() method.

Will return the output profile in the output.info['icc_profile'].

exception PIL.ImageCms.PyCMSError[source]
(pyCMS) Exception class. This is used for all errors in the pyCMS API.

Functions
PIL.ImageCms.applyTransform(im, transform, inPlace=False)[source]
(pyCMS) Applies a transform to a given image.

If im.mode != transform.inMode, a PyCMSError is raised.

If inPlace is True and transform.inMode != transform.outMode, a PyCMSError is raised.

If im.mode, transform.inMode or transform.outMode is not supported by pyCMSdll or the profiles you used for the transform, a PyCMSError is raised.

If an error occurs while the transform is being applied, a PyCMSError is raised.

This function applies a pre-calculated transform (from ImageCms.buildTransform() or ImageCms.buildTransformFromOpenProfiles()) to an image. The transform can be used for multiple images, saving considerable calculation time if doing the same conversion multiple times.

If you want to modify im in-place instead of receiving a new image as the return value, set inPlace to True. This can only be done if transform.inMode and transform.outMode are the same, because we can’t change the mode in-place (the buffer sizes for some modes are different). The default behavior is to return a new Image object of the same dimensions in mode transform.outMode.

PARAMETERS:
im – An Image object, and im.mode must be the same as the inMode supported by the transform.

transform – A valid CmsTransform class object

inPlace – Bool. If True, im is modified in place and None is returned, if False, a new Image object with the transform applied is returned (and im is not changed). The default is False.

RETURNS:
Either None, or a new Image object, depending on the value of inPlace. The profile will be returned in the image’s info['icc_profile'].

RAISES:
PyCMSError –

PIL.ImageCms.buildProofTransform(inputProfile, outputProfile, proofProfile, inMode, outMode, renderingIntent=Intent.PERCEPTUAL, proofRenderingIntent=Intent.ABSOLUTE_COLORIMETRIC, flags=16384)[source]
(pyCMS) Builds an ICC transform mapping from the inputProfile to the outputProfile, but tries to simulate the result that would be obtained on the proofProfile device.

If the input, output, or proof profiles specified are not valid filenames, a PyCMSError will be raised.

If an error occurs during creation of the transform, a PyCMSError will be raised.

If inMode or outMode are not a mode supported by the outputProfile (or by pyCMS), a PyCMSError will be raised.

This function builds and returns an ICC transform from the inputProfile to the outputProfile, but tries to simulate the result that would be obtained on the proofProfile device using renderingIntent and proofRenderingIntent to determine what to do with out-of-gamut colors. This is known as “soft-proofing”. It will ONLY work for converting images that are in inMode to images that are in outMode color format (PIL mode, i.e. “RGB”, “RGBA”, “CMYK”, etc.).

Usage of the resulting transform object is exactly the same as with ImageCms.buildTransform().

Proof profiling is generally used when using an output device to get a good idea of what the final printed/displayed image would look like on the proofProfile device when it’s quicker and easier to use the output device for judging color. Generally, this means that the output device is a monitor, or a dye-sub printer (etc.), and the simulated device is something more expensive, complicated, or time consuming (making it difficult to make a real print for color judgement purposes).

Soft-proofing basically functions by adjusting the colors on the output device to match the colors of the device being simulated. However, when the simulated device has a much wider gamut than the output device, you may obtain marginal results.

PARAMETERS:
inputProfile – String, as a valid filename path to the ICC input profile you wish to use for this transform, or a profile object

outputProfile – String, as a valid filename path to the ICC output (monitor, usually) profile you wish to use for this transform, or a profile object

proofProfile – String, as a valid filename path to the ICC proof profile you wish to use for this transform, or a profile object

inMode – String, as a valid PIL mode that the appropriate profile also supports (i.e. “RGB”, “RGBA”, “CMYK”, etc.)

outMode – String, as a valid PIL mode that the appropriate profile also supports (i.e. “RGB”, “RGBA”, “CMYK”, etc.)

renderingIntent –

Integer (0-3) specifying the rendering intent you wish to use for the input->proof (simulated) transform

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what they do.

proofRenderingIntent –

Integer (0-3) specifying the rendering intent you wish to use for proof->output transform

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what they do.

flags – Integer (0-…) specifying additional flags

RETURNS:
A CmsTransform class object.

RAISES:
PyCMSError –

PIL.ImageCms.buildProofTransformFromOpenProfiles(inputProfile, outputProfile, proofProfile, inMode, outMode, renderingIntent=Intent.PERCEPTUAL, proofRenderingIntent=Intent.ABSOLUTE_COLORIMETRIC, flags=16384)
(pyCMS) Builds an ICC transform mapping from the inputProfile to the outputProfile, but tries to simulate the result that would be obtained on the proofProfile device.

If the input, output, or proof profiles specified are not valid filenames, a PyCMSError will be raised.

If an error occurs during creation of the transform, a PyCMSError will be raised.

If inMode or outMode are not a mode supported by the outputProfile (or by pyCMS), a PyCMSError will be raised.

This function builds and returns an ICC transform from the inputProfile to the outputProfile, but tries to simulate the result that would be obtained on the proofProfile device using renderingIntent and proofRenderingIntent to determine what to do with out-of-gamut colors. This is known as “soft-proofing”. It will ONLY work for converting images that are in inMode to images that are in outMode color format (PIL mode, i.e. “RGB”, “RGBA”, “CMYK”, etc.).

Usage of the resulting transform object is exactly the same as with ImageCms.buildTransform().

Proof profiling is generally used when using an output device to get a good idea of what the final printed/displayed image would look like on the proofProfile device when it’s quicker and easier to use the output device for judging color. Generally, this means that the output device is a monitor, or a dye-sub printer (etc.), and the simulated device is something more expensive, complicated, or time consuming (making it difficult to make a real print for color judgement purposes).

Soft-proofing basically functions by adjusting the colors on the output device to match the colors of the device being simulated. However, when the simulated device has a much wider gamut than the output device, you may obtain marginal results.

PARAMETERS:
inputProfile – String, as a valid filename path to the ICC input profile you wish to use for this transform, or a profile object

outputProfile – String, as a valid filename path to the ICC output (monitor, usually) profile you wish to use for this transform, or a profile object

proofProfile – String, as a valid filename path to the ICC proof profile you wish to use for this transform, or a profile object

inMode – String, as a valid PIL mode that the appropriate profile also supports (i.e. “RGB”, “RGBA”, “CMYK”, etc.)

outMode – String, as a valid PIL mode that the appropriate profile also supports (i.e. “RGB”, “RGBA”, “CMYK”, etc.)

renderingIntent –

Integer (0-3) specifying the rendering intent you wish to use for the input->proof (simulated) transform

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what they do.

proofRenderingIntent –

Integer (0-3) specifying the rendering intent you wish to use for proof->output transform

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what they do.

flags – Integer (0-…) specifying additional flags

RETURNS:
A CmsTransform class object.

RAISES:
PyCMSError –

PIL.ImageCms.buildTransform(inputProfile, outputProfile, inMode, outMode, renderingIntent=Intent.PERCEPTUAL, flags=0)[source]
(pyCMS) Builds an ICC transform mapping from the inputProfile to the outputProfile. Use applyTransform to apply the transform to a given image.

If the input or output profiles specified are not valid filenames, a PyCMSError will be raised. If an error occurs during creation of the transform, a PyCMSError will be raised.

If inMode or outMode are not a mode supported by the outputProfile (or by pyCMS), a PyCMSError will be raised.

This function builds and returns an ICC transform from the inputProfile to the outputProfile using the renderingIntent to determine what to do with out-of-gamut colors. It will ONLY work for converting images that are in inMode to images that are in outMode color format (PIL mode, i.e. “RGB”, “RGBA”, “CMYK”, etc.).

Building the transform is a fair part of the overhead in ImageCms.profileToProfile(), so if you’re planning on converting multiple images using the same input/output settings, this can save you time. Once you have a transform object, it can be used with ImageCms.applyProfile() to convert images without the need to re-compute the lookup table for the transform.

The reason pyCMS returns a class object rather than a handle directly to the transform is that it needs to keep track of the PIL input/output modes that the transform is meant for. These attributes are stored in the inMode and outMode attributes of the object (which can be manually overridden if you really want to, but I don’t know of any time that would be of use, or would even work).

PARAMETERS:
inputProfile – String, as a valid filename path to the ICC input profile you wish to use for this transform, or a profile object

outputProfile – String, as a valid filename path to the ICC output profile you wish to use for this transform, or a profile object

inMode – String, as a valid PIL mode that the appropriate profile also supports (i.e. “RGB”, “RGBA”, “CMYK”, etc.)

outMode – String, as a valid PIL mode that the appropriate profile also supports (i.e. “RGB”, “RGBA”, “CMYK”, etc.)

renderingIntent –

Integer (0-3) specifying the rendering intent you wish to use for the transform

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what they do.

flags – Integer (0-…) specifying additional flags

RETURNS:
A CmsTransform class object.

RAISES:
PyCMSError –

PIL.ImageCms.buildTransformFromOpenProfiles(inputProfile, outputProfile, inMode, outMode, renderingIntent=Intent.PERCEPTUAL, flags=0)
(pyCMS) Builds an ICC transform mapping from the inputProfile to the outputProfile. Use applyTransform to apply the transform to a given image.

If the input or output profiles specified are not valid filenames, a PyCMSError will be raised. If an error occurs during creation of the transform, a PyCMSError will be raised.

If inMode or outMode are not a mode supported by the outputProfile (or by pyCMS), a PyCMSError will be raised.

This function builds and returns an ICC transform from the inputProfile to the outputProfile using the renderingIntent to determine what to do with out-of-gamut colors. It will ONLY work for converting images that are in inMode to images that are in outMode color format (PIL mode, i.e. “RGB”, “RGBA”, “CMYK”, etc.).

Building the transform is a fair part of the overhead in ImageCms.profileToProfile(), so if you’re planning on converting multiple images using the same input/output settings, this can save you time. Once you have a transform object, it can be used with ImageCms.applyProfile() to convert images without the need to re-compute the lookup table for the transform.

The reason pyCMS returns a class object rather than a handle directly to the transform is that it needs to keep track of the PIL input/output modes that the transform is meant for. These attributes are stored in the inMode and outMode attributes of the object (which can be manually overridden if you really want to, but I don’t know of any time that would be of use, or would even work).

PARAMETERS:
inputProfile – String, as a valid filename path to the ICC input profile you wish to use for this transform, or a profile object

outputProfile – String, as a valid filename path to the ICC output profile you wish to use for this transform, or a profile object

inMode – String, as a valid PIL mode that the appropriate profile also supports (i.e. “RGB”, “RGBA”, “CMYK”, etc.)

outMode – String, as a valid PIL mode that the appropriate profile also supports (i.e. “RGB”, “RGBA”, “CMYK”, etc.)

renderingIntent –

Integer (0-3) specifying the rendering intent you wish to use for the transform

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what they do.

flags – Integer (0-…) specifying additional flags

RETURNS:
A CmsTransform class object.

RAISES:
PyCMSError –

PIL.ImageCms.createProfile(colorSpace, colorTemp=-1)[source]
(pyCMS) Creates a profile.

If colorSpace not in ["LAB", "XYZ", "sRGB"], a PyCMSError is raised.

If using LAB and colorTemp is not a positive integer, a PyCMSError is raised.

If an error occurs while creating the profile, a PyCMSError is raised.

Use this function to create common profiles on-the-fly instead of having to supply a profile on disk and knowing the path to it. It returns a normal CmsProfile object that can be passed to ImageCms.buildTransformFromOpenProfiles() to create a transform to apply to images.

PARAMETERS:
colorSpace – String, the color space of the profile you wish to create. Currently only “LAB”, “XYZ”, and “sRGB” are supported.

colorTemp – Positive integer for the white point for the profile, in degrees Kelvin (i.e. 5000, 6500, 9600, etc.). The default is for D50 illuminant if omitted (5000k). colorTemp is ONLY applied to LAB profiles, and is ignored for XYZ and sRGB.

RETURNS:
A CmsProfile class object

RAISES:
PyCMSError –

PIL.ImageCms.getDefaultIntent(profile)[source]
(pyCMS) Gets the default intent name for the given profile.

If profile isn’t a valid CmsProfile object or filename to a profile, a PyCMSError is raised.

If an error occurs while trying to obtain the default intent, a PyCMSError is raised.

Use this function to determine the default (and usually best optimized) rendering intent for this profile. Most profiles support multiple rendering intents, but are intended mostly for one type of conversion. If you wish to use a different intent than returned, use ImageCms.isIntentSupported() to verify it will work first.

PARAMETERS:
profile – EITHER a valid CmsProfile object, OR a string of the filename of an ICC profile.

RETURNS:
Integer 0-3 specifying the default rendering intent for this profile.

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what
they do.

RAISES:
PyCMSError –

PIL.ImageCms.getOpenProfile(profileFilename)[source]
(pyCMS) Opens an ICC profile file.

The PyCMSProfile object can be passed back into pyCMS for use in creating transforms and such (as in ImageCms.buildTransformFromOpenProfiles()).

If profileFilename is not a valid filename for an ICC profile, a PyCMSError will be raised.

PARAMETERS:
profileFilename – String, as a valid filename path to the ICC profile you wish to open, or a file-like object.

RETURNS:
A CmsProfile class object.

RAISES:
PyCMSError –

PIL.ImageCms.getProfileCopyright(profile)[source]
(pyCMS) Gets the copyright for the given profile.

If profile isn’t a valid CmsProfile object or filename to a profile, a PyCMSError is raised.

If an error occurs while trying to obtain the copyright tag, a PyCMSError is raised.

Use this function to obtain the information stored in the profile’s copyright tag.

PARAMETERS:
profile – EITHER a valid CmsProfile object, OR a string of the filename of an ICC profile.

RETURNS:
A string containing the internal profile information stored in an ICC tag.

RAISES:
PyCMSError –

PIL.ImageCms.getProfileDescription(profile)[source]
(pyCMS) Gets the description for the given profile.

If profile isn’t a valid CmsProfile object or filename to a profile, a PyCMSError is raised.

If an error occurs while trying to obtain the description tag, a PyCMSError is raised.

Use this function to obtain the information stored in the profile’s description tag.

PARAMETERS:
profile – EITHER a valid CmsProfile object, OR a string of the filename of an ICC profile.

RETURNS:
A string containing the internal profile information stored in an ICC tag.

RAISES:
PyCMSError –

PIL.ImageCms.getProfileInfo(profile)[source]
(pyCMS) Gets the internal product information for the given profile.

If profile isn’t a valid CmsProfile object or filename to a profile, a PyCMSError is raised.

If an error occurs while trying to obtain the info tag, a PyCMSError is raised.

Use this function to obtain the information stored in the profile’s info tag. This often contains details about the profile, and how it was created, as supplied by the creator.

PARAMETERS:
profile – EITHER a valid CmsProfile object, OR a string of the filename of an ICC profile.

RETURNS:
A string containing the internal profile information stored in an ICC tag.

RAISES:
PyCMSError –

PIL.ImageCms.getProfileManufacturer(profile)[source]
(pyCMS) Gets the manufacturer for the given profile.

If profile isn’t a valid CmsProfile object or filename to a profile, a PyCMSError is raised.

If an error occurs while trying to obtain the manufacturer tag, a PyCMSError is raised.

Use this function to obtain the information stored in the profile’s manufacturer tag.

PARAMETERS:
profile – EITHER a valid CmsProfile object, OR a string of the filename of an ICC profile.

RETURNS:
A string containing the internal profile information stored in an ICC tag.

RAISES:
PyCMSError –

PIL.ImageCms.getProfileModel(profile)[source]
(pyCMS) Gets the model for the given profile.

If profile isn’t a valid CmsProfile object or filename to a profile, a PyCMSError is raised.

If an error occurs while trying to obtain the model tag, a PyCMSError is raised.

Use this function to obtain the information stored in the profile’s model tag.

PARAMETERS:
profile – EITHER a valid CmsProfile object, OR a string of the filename of an ICC profile.

RETURNS:
A string containing the internal profile information stored in an ICC tag.

RAISES:
PyCMSError –

PIL.ImageCms.getProfileName(profile)[source]
(pyCMS) Gets the internal product name for the given profile.

If profile isn’t a valid CmsProfile object or filename to a profile, a PyCMSError is raised If an error occurs while trying to obtain the name tag, a PyCMSError is raised.

Use this function to obtain the INTERNAL name of the profile (stored in an ICC tag in the profile itself), usually the one used when the profile was originally created. Sometimes this tag also contains additional information supplied by the creator.

PARAMETERS:
profile – EITHER a valid CmsProfile object, OR a string of the filename of an ICC profile.

RETURNS:
A string containing the internal name of the profile as stored in an ICC tag.

RAISES:
PyCMSError –

PIL.ImageCms.get_display_profile(handle=None)[source]
(experimental) Fetches the profile for the current display device.

RETURNS:
None if the profile is not known.

PIL.ImageCms.isIntentSupported(profile, intent, direction)[source]
(pyCMS) Checks if a given intent is supported.

Use this function to verify that you can use your desired intent with profile, and that profile can be used for the input/output/proof profile as you desire.

Some profiles are created specifically for one “direction”, can cannot be used for others. Some profiles can only be used for certain rendering intents, so it’s best to either verify this before trying to create a transform with them (using this function), or catch the potential PyCMSError that will occur if they don’t support the modes you select.

PARAMETERS:
profile – EITHER a valid CmsProfile object, OR a string of the filename of an ICC profile.

intent –

Integer (0-3) specifying the rendering intent you wish to use with this profile

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what
they do.

direction –

Integer specifying if the profile is to be used for input, output, or proof

INPUT = 0 (or use ImageCms.Direction.INPUT) OUTPUT = 1 (or use ImageCms.Direction.OUTPUT) PROOF = 2 (or use ImageCms.Direction.PROOF)

RETURNS:
1 if the intent/direction are supported, -1 if they are not.

RAISES:
PyCMSError –

PIL.ImageCms.profileToProfile(im, inputProfile, outputProfile, renderingIntent=Intent.PERCEPTUAL, outputMode=None, inPlace=False, flags=0)[source]
(pyCMS) Applies an ICC transformation to a given image, mapping from inputProfile to outputProfile.

If the input or output profiles specified are not valid filenames, a PyCMSError will be raised. If inPlace is True and outputMode != im.mode, a PyCMSError will be raised. If an error occurs during application of the profiles, a PyCMSError will be raised. If outputMode is not a mode supported by the outputProfile (or by pyCMS), a PyCMSError will be raised.

This function applies an ICC transformation to im from inputProfile’s color space to outputProfile’s color space using the specified rendering intent to decide how to handle out-of-gamut colors.

outputMode can be used to specify that a color mode conversion is to be done using these profiles, but the specified profiles must be able to handle that mode. I.e., if converting im from RGB to CMYK using profiles, the input profile must handle RGB data, and the output profile must handle CMYK data.

PARAMETERS:
im – An open Image object (i.e. Image.new(…) or Image.open(…), etc.)

inputProfile – String, as a valid filename path to the ICC input profile you wish to use for this image, or a profile object

outputProfile – String, as a valid filename path to the ICC output profile you wish to use for this image, or a profile object

renderingIntent –

Integer (0-3) specifying the rendering intent you wish to use for the transform

ImageCms.Intent.PERCEPTUAL = 0 (DEFAULT) ImageCms.Intent.RELATIVE_COLORIMETRIC = 1 ImageCms.Intent.SATURATION = 2 ImageCms.Intent.ABSOLUTE_COLORIMETRIC = 3

see the pyCMS documentation for details on rendering intents and what they do.

outputMode – A valid PIL mode for the output image (i.e. “RGB”, “CMYK”, etc.). Note: if rendering the image “inPlace”, outputMode MUST be the same mode as the input, or omitted completely. If omitted, the outputMode will be the same as the mode of the input image (im.mode)

inPlace – Boolean. If True, the original image is modified in-place, and None is returned. If False (default), a new Image object is returned with the transform applied.

flags – Integer (0-…) specifying additional flags

RETURNS:
Either None or a new Image object, depending on the value of inPlace

RAISES:
PyCMSError –

PIL.ImageCms.versions()[source]
(pyCMS) Fetches versions.

CmsProfile
The ICC color profiles are wrapped in an instance of the class CmsProfile. The specification ICC.1:2010 contains more information about the meaning of the values in ICC profiles.

For convenience, all XYZ-values are also given as xyY-values (so they can be easily displayed in a chromaticity diagram, for example).

class PIL.ImageCms.CmsProfile
creation_date: Optional[datetime.datetime]
Date and time this profile was first created (see 7.2.1 of ICC.1:2010).

version: float
The version number of the ICC standard that this profile follows (e.g. 2.0).

icc_version: int
Same as version, but in encoded format (see 7.2.4 of ICC.1:2010).

device_class: str
4-character string identifying the profile class. One of scnr, mntr, prtr, link, spac, abst, nmcl (see 7.2.5 of ICC.1:2010 for details).

xcolor_space: str
4-character string (padded with whitespace) identifying the color space, e.g. XYZ␣, RGB␣ or CMYK (see 7.2.6 of ICC.1:2010 for details).

connection_space: str
4-character string (padded with whitespace) identifying the color space on the B-side of the transform (see 7.2.7 of ICC.1:2010 for details).

header_flags: int
The encoded header flags of the profile (see 7.2.11 of ICC.1:2010 for details).

header_manufacturer: str
4-character string (padded with whitespace) identifying the device manufacturer, which shall match the signature contained in the appropriate section of the ICC signature registry found at www.color.org (see 7.2.12 of ICC.1:2010).

header_model: str
4-character string (padded with whitespace) identifying the device model, which shall match the signature contained in the appropriate section of the ICC signature registry found at www.color.org (see 7.2.13 of ICC.1:2010).

attributes: int
Flags used to identify attributes unique to the particular device setup for which the profile is applicable (see 7.2.14 of ICC.1:2010 for details).

rendering_intent: int
The rendering intent to use when combining this profile with another profile (usually overridden at run-time, but provided here for DeviceLink and embedded source profiles, see 7.2.15 of ICC.1:2010).

One of ImageCms.Intent.ABSOLUTE_COLORIMETRIC, ImageCms.Intent.PERCEPTUAL, ImageCms.Intent.RELATIVE_COLORIMETRIC and ImageCms.Intent.SATURATION.

profile_id: bytes
A sequence of 16 bytes identifying the profile (via a specially constructed MD5 sum), or 16 binary zeroes if the profile ID has not been calculated (see 7.2.18 of ICC.1:2010).

copyright: Optional[str]
The text copyright information for the profile (see 9.2.21 of ICC.1:2010).

manufacturer: Optional[str]
The (English) display string for the device manufacturer (see 9.2.22 of ICC.1:2010).

model: Optional[str]
The (English) display string for the device model of the device for which this profile is created (see 9.2.23 of ICC.1:2010).

profile_description: Optional[str]
The (English) display string for the profile description (see 9.2.41 of ICC.1:2010).

target: Optional[str]
The name of the registered characterization data set, or the measurement data for a characterization target (see 9.2.14 of ICC.1:2010).

red_colorant: Optional[tuple[tuple[float]]]
The first column in the matrix used in matrix/TRC transforms (see 9.2.44 of ICC.1:2010).

The value is in the format ((X, Y, Z), (x, y, Y)), if available.

green_colorant: Optional[tuple[tuple[float]]]
The second column in the matrix used in matrix/TRC transforms (see 9.2.30 of ICC.1:2010).

The value is in the format ((X, Y, Z), (x, y, Y)), if available.

blue_colorant: Optional[tuple[tuple[float]]]
The third column in the matrix used in matrix/TRC transforms (see 9.2.4 of ICC.1:2010).

The value is in the format ((X, Y, Z), (x, y, Y)), if available.

luminance: Optional[tuple[tuple[float]]]
The absolute luminance of emissive devices in candelas per square metre as described by the Y channel (see 9.2.32 of ICC.1:2010).

The value is in the format ((X, Y, Z), (x, y, Y)), if available.

chromaticity: Optional[tuple[tuple[float]]]
The data of the phosphor/colorant chromaticity set used (red, green and blue channels, see 9.2.16 of ICC.1:2010).

The value is in the format ((x, y, Y), (x, y, Y), (x, y, Y)), if available.

chromatic_adaption: tuple[tuple[float]]
The chromatic adaption matrix converts a color measured using the actual illumination conditions and relative to the actual adopted white, to a color relative to the PCS adopted white, with complete adaptation from the actual adopted white chromaticity to the PCS adopted white chromaticity (see 9.2.15 of ICC.1:2010).

Two 3-tuples of floats are returned in a 2-tuple, one in (X, Y, Z) space and one in (x, y, Y) space.

colorant_table: list[str]
This tag identifies the colorants used in the profile by a unique name and set of PCSXYZ or PCSLAB values (see 9.2.19 of ICC.1:2010).

colorant_table_out: list[str]
This tag identifies the colorants used in the profile by a unique name and set of PCSLAB values (for DeviceLink profiles only, see 9.2.19 of ICC.1:2010).

colorimetric_intent: Optional[str]
4-character string (padded with whitespace) identifying the image state of PCS colorimetry produced using the colorimetric intent transforms (see 9.2.20 of ICC.1:2010 for details).

perceptual_rendering_intent_gamut: Optional[str]
4-character string (padded with whitespace) identifying the (one) standard reference medium gamut (see 9.2.37 of ICC.1:2010 for details).

saturation_rendering_intent_gamut: Optional[str]
4-character string (padded with whitespace) identifying the (one) standard reference medium gamut (see 9.2.37 of ICC.1:2010 for details).

technology: Optional[str]
4-character string (padded with whitespace) identifying the device technology (see 9.2.47 of ICC.1:2010 for details).

media_black_point: Optional[tuple[tuple[float]]]
This tag specifies the media black point and is used for generating absolute colorimetry.

This tag was available in ICC 3.2, but it is removed from version 4.

The value is in the format ((X, Y, Z), (x, y, Y)), if available.

media_white_point_temperature: Optional[float]
Calculates the white point temperature (see the LCMS documentation for more information).

viewing_condition: Optional[str]
The (English) display string for the viewing conditions (see 9.2.48 of ICC.1:2010).

screening_description: Optional[str]
The (English) display string for the screening conditions.

This tag was available in ICC 3.2, but it is removed from version 4.

red_primary: Optional[tuple[tuple[float]]]
The XYZ-transformed of the RGB primary color red (1, 0, 0).

The value is in the format ((X, Y, Z), (x, y, Y)), if available.

green_primary: Optional[tuple[tuple[float]]]
The XYZ-transformed of the RGB primary color green (0, 1, 0).

The value is in the format ((X, Y, Z), (x, y, Y)), if available.

blue_primary: Optional[tuple[tuple[float]]]
The XYZ-transformed of the RGB primary color blue (0, 0, 1).

The value is in the format ((X, Y, Z), (x, y, Y)), if available.

is_matrix_shaper: bool
True if this profile is implemented as a matrix shaper (see documentation on LCMS).

clut: dict[tuple[bool]]
Returns a dictionary of all supported intents and directions for the CLUT model.

The dictionary is indexed by intents (ImageCms.Intent.ABSOLUTE_COLORIMETRIC, ImageCms.Intent.PERCEPTUAL, ImageCms.Intent.RELATIVE_COLORIMETRIC and ImageCms.Intent.SATURATION).

The values are 3-tuples indexed by directions (ImageCms.Direction.INPUT, ImageCms.Direction.OUTPUT, ImageCms.Direction.PROOF).

The elements of the tuple are booleans. If the value is True, that intent is supported for that direction.

intent_supported: dict[tuple[bool]]
Returns a dictionary of all supported intents and directions.

The dictionary is indexed by intents (ImageCms.Intent.ABSOLUTE_COLORIMETRIC, ImageCms.Intent.PERCEPTUAL, ImageCms.Intent.RELATIVE_COLORIMETRIC and ImageCms.Intent.SATURATION).

The values are 3-tuples indexed by directions (ImageCms.Direction.INPUT, ImageCms.Direction.OUTPUT, ImageCms.Direction.PROOF).

The elements of the tuple are booleans. If the value is True, that intent is supported for that direction.

There is one function defined on the class:

is_intent_supported(intent, direction)
Returns if the intent is supported for the given direction.

Note that you can also get this information for all intents and directions with intent_supported.

PARAMETERS:
intent – One of ImageCms.Intent.ABSOLUTE_COLORIMETRIC, ImageCms.Intent.PERCEPTUAL, ImageCms.Intent.RELATIVE_COLORIMETRIC and ImageCms.Intent.SATURATION.

direction – One of ImageCms.Direction.INPUT, ImageCms.Direction.OUTPUT and ImageCms.Direction.PROOF

RETURNS:
Boolean if the intent and direction is supported.




ImageColor Module
The ImageColor module contains color tables and converters from CSS3-style color specifiers to RGB tuples. This module is used by PIL.Image.new() and the ImageDraw module, among others.

Color Names
The ImageColor module supports the following string formats:

Hexadecimal color specifiers, given as #rgb, #rgba, #rrggbb or #rrggbbaa, where r is red, g is green, b is blue and a is alpha (also called ‘opacity’). For example, #ff0000 specifies pure red, and #ff0000cc specifies red with 80% opacity (cc is 204 in decimal form, and 204 / 255 = 0.8).

RGB functions, given as rgb(red, green, blue) where the color values are integers in the range 0 to 255. Alternatively, the color values can be given as three percentages (0% to 100%). For example, rgb(255,0,0) and rgb(100%,0%,0%) both specify pure red.

Hue-Saturation-Lightness (HSL) functions, given as hsl(hue, saturation%, lightness%) where hue is the color given as an angle between 0 and 360 (red=0, green=120, blue=240), saturation is a value between 0% and 100% (gray=0%, full color=100%), and lightness is a value between 0% and 100% (black=0%, normal=50%, white=100%). For example, hsl(0,100%,50%) is pure red.

Hue-Saturation-Value (HSV) functions, given as hsv(hue, saturation%, value%) where hue and saturation are the same as HSL, and value is between 0% and 100% (black=0%, normal=100%). For example, hsv(0,100%,100%) is pure red. This format is also known as Hue-Saturation-Brightness (HSB), and can be given as hsb(hue, saturation%, brightness%), where each of the values are used as they are in HSV.

Common HTML color names. The ImageColor module provides some 140 standard color names, based on the colors supported by the X Window system and most web browsers. color names are case insensitive. For example, red and Red both specify pure red.

Functions
PIL.ImageColor.getrgb(color)[source]
Convert a color string to an RGB tuple. If the string cannot be parsed, this function raises a ValueError exception.

New in version 1.1.4.

PIL.ImageColor.getcolor(color, mode)[source]
Same as getrgb(), but converts the RGB value to a greyscale value if the mode is not color or a palette image. If the string cannot be parsed, this function raises a ValueError exception.

New in version 1.1.4.


ImageMath Module
The ImageMath module can be used to evaluate “image expressions”. The module provides a single eval() function, which takes an expression string and one or more images.

Example: Using the ImageMath module
from PIL import Image, ImageMath

with Image.open("image1.jpg") as im1:
    with Image.open("image2.jpg") as im2:

        out = ImageMath.eval("convert(min(a, b), 'L')", a=im1, b=im2)
        out.save("result.png")
PIL.ImageMath.eval(expression, environment)[source]
Evaluate expression in the given environment.

In the current version, ImageMath only supports single-layer images. To process multi-band images, use the split() method or merge() function.

PARAMETERS:
expression – A string which uses the standard Python expression syntax. In addition to the standard operators, you can also use the functions described below.

environment – A dictionary that maps image names to Image instances. You can use one or more keyword arguments instead of a dictionary, as shown in the above example. Note that the names must be valid Python identifiers.

RETURNS:
An image, an integer value, a floating point value, or a pixel tuple, depending on the expression.

Expression syntax
Expressions are standard Python expressions, but they’re evaluated in a non-standard environment. You can use PIL methods as usual, plus the following set of operators and functions:

Standard Operators
You can use standard arithmetical operators for addition (+), subtraction (-), multiplication (*), and division (/).

The module also supports unary minus (-), modulo (%), and power (**) operators.

Note that all operations are done with 32-bit integers or 32-bit floating point values, as necessary. For example, if you add two 8-bit images, the result will be a 32-bit integer image. If you add a floating point constant to an 8-bit image, the result will be a 32-bit floating point image.

You can force conversion using the convert(), float(), and int() functions described below.

Bitwise Operators
The module also provides operations that operate on individual bits. This includes and (&), or (|), and exclusive or (^). You can also invert (~) all pixel bits.

Note that the operands are converted to 32-bit signed integers before the bitwise operation is applied. This means that you’ll get negative values if you invert an ordinary greyscale image. You can use the and (&) operator to mask off unwanted bits.

Bitwise operators don’t work on floating point images.

Logical Operators
Logical operators like and, or, and not work on entire images, rather than individual pixels.

An empty image (all pixels zero) is treated as false. All other images are treated as true.

Note that and and or return the last evaluated operand, while not always returns a boolean value.

Built-in Functions
These functions are applied to each individual pixel.

abs(image)
Absolute value.

convert(image, mode)
Convert image to the given mode. The mode must be given as a string constant.

float(image)
Convert image to 32-bit floating point. This is equivalent to convert(image, “F”).

int(image)
Convert image to 32-bit integer. This is equivalent to convert(image, “I”).

Note that 1-bit and 8-bit images are automatically converted to 32-bit integers if necessary to get a correct result.

max(image1, image2)
Maximum value.

min(image1, image2)
Minimum value.

ImagePalette Module
The ImagePalette module contains a class of the same name to represent the color palette of palette mapped images.

Note

The ImagePalette class has several methods, but they are all marked as “experimental.” Read that as you will. The [source] link is there for a reason.

class PIL.ImagePalette.ImagePalette(mode='RGB', palette=None, size=0)[source]
Color palette for palette mapped images

PARAMETERS:
mode – The mode to use for the palette. See: Modes. Defaults to “RGB”

palette – An optional palette. If given, it must be a bytearray, an array or a list of ints between 0-255. The list must consist of all channels for one color followed by the next color (e.g. RGBRGBRGB). Defaults to an empty palette.

getcolor(color, image=None)[source]
Given an rgb tuple, allocate palette entry.

Warning

This method is experimental.

getdata()[source]
Get palette contents in format suitable for the low-level im.putpalette primitive.

Warning

This method is experimental.

save(fp)[source]
Save palette to text file.

Warning

This method is experimental.

tobytes()[source]
Convert palette to bytes.

Warning

This method is experimental.

tostring()
Convert palette to bytes.

Warning

This method is experimental.

ImagePath Module
The ImagePath module is used to store and manipulate 2-dimensional vector data. Path objects can be passed to the methods on the ImageDraw module.

class PIL.ImagePath.Path
A path object. The coordinate list can be any sequence object containing either 2-tuples [(x, y), …] or numeric values [x, y, …].

You can also create a path object from another path object.

In 1.1.6 and later, you can also pass in any object that implements Python’s buffer API. The buffer should provide read access, and contain C floats in machine byte order.

The path object implements most parts of the Python sequence interface, and behaves like a list of (x, y) pairs. You can use len(), item access, and slicing as usual. However, the current version does not support slice assignment, or item and slice deletion.

PARAMETERS:
xy – A sequence. The sequence can contain 2-tuples [(x, y), …] or a flat list of numbers [x, y, …].

PIL.ImagePath.Path.compact(distance=2)
Compacts the path, by removing points that are close to each other. This method modifies the path in place, and returns the number of points left in the path.

distance is measured as Manhattan distance and defaults to two pixels.

PIL.ImagePath.Path.getbbox()
Gets the bounding box of the path.

RETURNS:
(x0, y0, x1, y1)

PIL.ImagePath.Path.map(function)
Maps the path through a function.

PIL.ImagePath.Path.tolist(flat=0)
Converts the path to a Python list [(x, y), …].

PARAMETERS:
flat – By default, this function returns a list of 2-tuples [(x, y), …]. If this argument is True, it returns a flat list [x, y, …] instead.

RETURNS:
A list of coordinates. See flat.

PIL.ImagePath.Path.transform(matrix)
Transforms the path in place, using an affine transform. The matrix is a 6-tuple (a, b, c, d, e, f), and each point is mapped as follows:

xOut = xIn * a + yIn * b + c
yOut = xIn * d + yIn * e + f

ImageQt Module
The ImageQt module contains support for creating PyQt6, PySide6, PyQt5 or PySide2 QImage objects from PIL images.

Qt 5 reached end-of-life on 2020-12-08 for open-source users (and will reach EOL on 2023-12-08 for commercial licence holders).

Support for PyQt5 and PySide2 has been deprecated from ImageQt and will be removed in Pillow 10 (2023-07-01). Upgrade to PyQt6 or PySide6 instead.

New in version 1.1.6.

class PIL.ImageQt.ImageQt(image)
Creates an ImageQt object from a PIL Image object. This class is a subclass of QtGui.QImage, which means that you can pass the resulting objects directly to PyQt6/PySide6/PyQt5/PySide2 API functions and methods.

This operation is currently supported for mode 1, L, P, RGB, and RGBA images. To handle other modes, you need to convert the image first.



ImageSequence Module
The ImageSequence module contains a wrapper class that lets you iterate over the frames of an image sequence.

Extracting frames from an animation
from PIL import Image, ImageSequence

with Image.open("animation.fli") as im:
    index = 1
    for frame in ImageSequence.Iterator(im):
        frame.save(f"frame{index}.png")
        index += 1
The Iterator class
class PIL.ImageSequence.Iterator(im)[source]
This class implements an iterator object that can be used to loop over an image sequence.

You can use the [] operator to access elements by index. This operator will raise an IndexError if you try to access a nonexistent frame.

PARAMETERS:
im – An image object.

Functions
PIL.ImageSequence.all_frames(im, func=None)[source]
Applies a given function to all frames in an image or a list of images. The frames are returned as a list of separate images.

PARAMETERS:
im – An image, or a list of images.

func – The function to apply to all of the image frames.

RETURNS:
A list of images.
