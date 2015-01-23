# imgo
imgo is a tool for automated lossless image optimization. Run it in a folder with images to reduce file sizes to a minimum.

While developing this utility, we came to a conclusion that manual tuning of compression options might be quite effective sometimes; this is why imgo does not totally automate the optimization process. Instead, the tool provides a number of options you can try to get the best result, and automates image analysis to choose the best compression technique.

## Usage

`imgo .` Run imgo with a dot as a parameter (`.`) to process all images (png, gif and jpg files) that could be found in the current folder and its subfolders, recursively.

It is also possible to pass one or several file names and/or folder names:

* `imgo file.png`
* `imgo file.png file.jpg file.gif`
* `imgo file.png folder folder-2`

## Compression levels

For PNG file format, various compression rates that imgo provides can be categorized into 3 levels:

* `imgo file.png` — basic compression level (default), works relatively fast and applies only "safe" transformations. Recommended for images that must be displayed correctly by IE6.

* `imgo -b file.png` — advanced compression level (enabled with `-b` or `--brute` parameter), applies all the transformations available. Some resulting files (with transparency) may display incorrectly in IE6.

* `imgo -m -b file.png` — maximum compression level, applies compression multiple times to a single file (enabled with -m or -multipass parameter). Sometimes this saves you a few extra bytes (usually less than 1% of total file size, but hardly more).

## Other optimizer features

In addition to its main purpose (image compression), imgo provides a number of tools to fine-tune PNG image file format.

### Setting the bkgd chunk

The use of bKGD chunk is one of the graceful degradation techniques that makes pngfix hack unnecessary in IE6. The source of the problem is IE6 that incorrectly displays PNG24 images with alpha transparency.

This is an example HTML5 log with alpha transparency:

![Alpha-transparent HTML5 logo](http://img-fotki.yandex.ru/get/6101/1861097.22/0_79fd9_13413784_orig)

In IE6, this looks like this:

![alpha-transparent HTML5 logo in IE6](http://img-fotki.yandex.ru/get/6101/1861097.22/0_79dff_15ed20fb_orig)

While AlphaImageLoader filter is a well-known fix for that, it's not the only one. If we specify a color for a bKGD chunk, IE6 will show a picture as if layered over a background of that color.

`imgo -bkgd#ffffff file.png` Logo on a white background:

![Logo on a white background](http://img-fotki.yandex.ru/get/6200/1861097.22/0_79e00_9028cea8_orig)

`imgo -bkgd#ffc927 file.png` Logo on an orange background:

![Logo on an orange background](http://img-fotki.yandex.ru/get/6201/1861097.22/0_79e01_a0d043af_orig)

Note that we do not remove transparency; instead, we replace grayish color drawn by IE by a specific one. With AlphaImageLoader filter applied, an image would still support semi-transparency. Other browsers ignore the bKGD chunk and display alpha transparency as they should.

### Transparent area clearing

All PNG24 images with transparency go through the so-called "transparent area clearing" optimization.

Simply put, each png24 with alpha transparency can be represented by two layers; the first one is the image itself, the second one is the transparency mask.

Test image (19.6 Kb):

![Test image](http://img-fotki.yandex.ru/get/6101/1861097.22/0_79e02_dae7fe87_-1-orig)

When separated into two parts, they look like this:

The image itself (14.6 Kb):

![The image](http://img-fotki.yandex.ru/get/6200/1861097.22/0_79e03_ef2d41a7_orig)

The alpha channel (the mask, 1.16 Kb):

![The alpha channel](http://img-fotki.yandex.ru/get/6101/1861097.22/0_79e04_4c1907ac_orig)

The original image contains redundant data that affects its size. imgo fills these areas with black.

If we pass the original image through imgo and then separate it again, the image itself will look like this (3.27 Kb):

![Transparent areas clearing](http://img-fotki.yandex.ru/get/6200/1861097.22/0_79e05_7e9f41bb_-1-orig)

This method doesn't affect visible areas of the image, therefore this transformation is safe (i.e. lossless).

Sometimes you need just the image itself without any transparency; if so, specify the `-rt` parameter:

`imgo -rt file.png`

This will create a new file file_imgo_rt.png that is the same as the original one but with transparency layer removed.

### Converting to PNG8 with alpha transparency

PNG8 supports two types of transparency:

1. Binary transparency (a pixel is either completely transparent, or completely opaque). The same kind of transparency is supported by a GIF format. Binary transparency is most commonly used with PNG8.

2. Alpha channel (a pixel can have any transparency on the scale from 0 to 255, with step 1).

imgo allows for converting png24 with an alpha channel to png8 with an alpha channel. It's worth mentioning that png8 is limited by a 256 color palette, therefore color reduction will occur if the source image has many colors.

HTML5 logo: PNG24 + Alpha channel (7,1 Kb)

![PNG24 + Alpha channel](http://img-fotki.yandex.ru/get/6201/1861097.22/0_79fda_e54f08b0_orig)

HTML5 Logo: PNG8 + Alpha channel (3.2 Kb)

![PNG8 + Alpha channel](http://img-fotki.yandex.ru/get/6200/1861097.22/0_79e07_75eee8fc_orig)

In addition to lower file size, PNG8 transparency is also a graceful degradation technique. The point is that IE6 doesn't show these images correctly, displaying all semi-transparent areas as fully transparent ones.

`imgo -png8a file.png` After running this command, another file named file_png8a.png appears near the original file.png. You can compare these two and see if the reduction in quality is a reasonable trade-off for getting a smaller file.

### Decomposing PNG24+Alpha into two files

One of the possible ways to reduce file size of such images is decomposing them into two files.

We have a test image (7.1 Kb):

![Test image](http://img-fotki.yandex.ru/get/5606/1861097.22/0_79fdb_10d825bc_orig)

By running `imgo -s file.png` , we get the following two files:

The first one contains all opaque pixels (1.61 Kb):

![All opaque pixels](http://img-fotki.yandex.ru/get/6101/1861097.22/0_79e09_d99329a3_orig)

The second one contains all the semi-transparency (1.28 Kb):

![All the semi-transparency](http://img-fotki.yandex.ru/get/6201/1861097.22/0_79e0a_fe97ab56_orig)

By positioning these images one over the other, we get the original image.

After running imgo -s file.png , the following 3 files are created near the original one:

* `file_imgo_alpha.png` — PNG24, a layer with semi-transparent pixels only;
* `file_imgo_crop.png` — PNG24, a layer with opaque pixels only;
* `file_imgo_crop-nq8.png` — PNG8 made from `file_imgo_crop.png` (a lossy conversion)

In most cases, you win in overall file size by combining a semi-transparent layer and PNG8. A separate PNG24 layer is provided mostly for visual comparison between PNG24 and PNG8 files: if the difference is subtle, this method is applicable to a particular image.

## Downloading and installing

Repository: http://github.com/imgo/imgo

Installation manual (MacOS): https://github.com/imgo/imgo#readme

Report bugs here: https://github.com/imgo/imgo/issues 

We are monitoring new image compression utilities and techniques and embed the best methods into new versions of imgo.

Subscribe to our github commits to stay in sync with the latest imgo updates.
