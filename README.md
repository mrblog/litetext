# litetext #

This small package (less than 1000 lines of Java code) was developed to provide a small and simple package for rendering text into an image (bitmap). It was developed for Google App Engine use where AWT and BufferedImage et al do not exist. This small utility can be used to render text on App Engine within the constraints of the JRE Class White List. This is derived from the "pbmtext" utility from NETpbm Copyright (C) 1991 by Jef Poskanzer.

These are crude black-and-white bitmap fonts - no anti-aliasing. A small number of BDF fonts in fixed-sizes are bundled into the package: Courier, Lucida-Bold-Italic, Lucida-Medium, Lucida-Medium-Italic, LucidaBright-DemiBold, LucidaTypewriter, LucidaTypewriter-Bold, and a default serif font.

The following code snippet demonstrates use of litetext:

```
    byte[] bmp_data = doRender("Render this text into a bitmap");
    // bmp_data contains a BMP file
```

## Usage ##

This package provides a simple text rendering capability. In its simplest form, it can be used like this:

```

 import org.toyz.litetext.FontUtils
 
 // ...
 
 byte[] bmp_data = doRender("Render this text into a bitmap");
 
 // bmp_data contains a BMP file
 
```

There is a working example Servlet in `examples/LiteTextServlet.java`

### Selecting Fonts ###

Select a font by specifying the font by name (String):

```
  byte[] bmp_data = doRender("Render this text into a bitmap using specified font", "Courier");
```

Available fonts are specified by name, as follows:

* Courier
* Lucida-Bold-Italic
* Lucida-Medium
* Lucida-Medium-Italic
* LucidaBright-DemiBold
* LucidaTypewriter
* LucidaTypewriter-Bold
* default

### Font Color ###

The font (foreground) color can be set with the setFgcolor(int[]) method, before calling doRender() e.g.:

```
  FontUtils fm = new FontUtils(); 
  // RGB color components for font (foreground) 
  color fgcolor[0] = 221; // #DDE0F4 (light gray) 
  fgcolor[1] = 224; 
  fgcolor[2] = 244; 
  fm.setFgcolor(fgcolor);
```

### Background Color/Gradient ###

The background is specified as a gradient, an array of RGB color components using `setGradient(int[][])`. For example, to set a single color background, supply a one-row array (one pixel) as shown below:

```
 int[][] graybg = new int[1][3]; 
 graybg[0][0] = 221; // #DDE0F4 (light gray) 
 graybg[0][1] = 224; 
 graybg[0][2] = 244; 
 fm.setGradient(graybg);
```

To use a gradient, supply more rows (pixels), which basically represent a one-pixel wide vertical bar of pixels (rows of RGB values). E.g.:

```
  int[][] gradar = new int[16][3]; 
  gradar[0][0] = 210; gradar[0][1] = 213; gradar[0][2] = 240;
  gradar[1][0] = 213; gradar[1][1] = 216; gradar[1][2] = 241;
  gradar[2][0] = 216; gradar[2][1] = 219; gradar[2][2] = 242;
  gradar[3][0] = 221; gradar[3][1] = 224; gradar[3][2] = 244;
  gradar[4][0] = 224; gradar[4][1] = 226; gradar[4][2] = 245;
  gradar[5][0] = 225; gradar[5][1] = 227; gradar[5][2] = 245;
  gradar[6][0] = 230; gradar[6][1] = 232; gradar[6][2] = 247;
  gradar[7][0] = 231; gradar[7][1] = 233; gradar[7][2] = 247;
  gradar[8][0] = 236; gradar[8][1] = 237; gradar[8][2] = 249;
  gradar[9][0] = 239; gradar[9][1] = 240; gradar[9][2] = 250;
  gradar[10][0] = 242; gradar[10][1] = 243; gradar[10][2] = 251;
  gradar[11][0] = 243; gradar[11][1] = 244; gradar[11][2] = 251;
  gradar[12][0] = 246; gradar[12][1] = 247; gradar[12][2] = 252;
  gradar[13][0] = 248; gradar[13][1] = 249; gradar[13][2] = 253;
  gradar[14][0] = 251; gradar[14][1] = 251; gradar[14][2] = 254;
  gradar[15][0] = 252; gradar[15][1] = 252; gradar[15][2] = 254;
  fm.setGradient(gradar);
```

## Ant Build ##
   
Type `ant jar` to create `build/litetext.jar`

## App Engine Java Images API ##

The resulting BMP file returned by `doRender()` can be passed to the App Engine Java Images API to produce, e.g. a JPEG image as follows:

```
 byte[] bmp_data = fm.doRender(inputText, fontname);

 com.google.appengine.api.images.ImagesService imagesService = ImagesServiceFactory.getImagesService();

 Image bmpImage = ImagesServiceFactory.makeImage(bmp_data);
 com.google.appengine.api.images.Transform flipit = ImagesServiceFactory.makeVerticalFlip();
 Image newImage = imagesService.applyTransform(flipit, bmpImage, com.google.appengine.api.images.ImagesService.OutputEncoding.JPEG);
```

### App Engine Limitations ###

The Images API limits requests to 1MB. This means that the resulting text image can be only about 500x500 pixels or so. Therefore, litetxt is restricted to a few short paragraphs of text per image on App Engine.

## Developer's Guide ##

The main class of litetext is:

* FontUtils

Other classes include:

* Font
* Glyph
* EndianUtils
* LittleEndianOutputStream

## Font Format Details ##

Fonts are simple bitmap fonts. Ultimately fonts are contained in the `Font` class.

Each printable character known by the font is stored in a `Glyph`, indexed by its ASCII code.

### Font Loading ###

Fonts are stuffed into the JAR and loaded from file-data into the above classes. See the `loadfont(String fontname)` method in `FontUtils` for more details.
   
### Font Files ###

The font data for a given font is stored in font files located in the JAR as `/fonts/{fontname}/*`

Each font has a `font.txt` file and N `glyph.*` files (`glyph.32 - glyph.255` one per ASCII printable character in this font). These files are Properties format compatible with the java.util.Properties class.

In addition, each glyph (printable character) has a corresponding file containing the bitmap for the character `glyph_bmap.*` a simple array of bytes, each representing a bit.
`
Additional bitmap fonts can be used by organizing the font data and bitmaps into these files in the proper format.

You may add them into the JAR by placing the new font data files into a `resources/font/{fontname}/` directory and running `ant jar`. Applications then select the font by name in `doRender()`.

Example font.txt: 
```
 maxwidth = 14
 maxheight = 20 
 x = -3 
 y = -5 
 oldfont = 0 
 fcols = 0 
 frows = 0
```

Example glyph.97 (lower-case 'a' data) file: 

```
 width = 7
 height = 7
 x = 1
 y = 0
 xadd = 9
```

Hexdump of corresponding glyph_bmap.97 (bitmap for lower-case 'a'): 

```
0000000 00 01 01 01 01 00 00 01 00 00 00 00 01 00 00 00 
0000010 00 00 00 01 00 00 01 01 01 01 01 00 01 00 00 00 
0000020 00 01 00 01 00 00 00 00 01 00 00 01 01 01 01 00 
0000030 01 
0000031
```

## Acknowledgements ##

We want to thank the following people and organizations for their help:

Jef Poskanzer who developed the pbmplus package.

>  
>  Copyright (C) 1989, 1991 by Jef Poskanzer.
>  
>  Permission to use, copy, modify, and distribute this software and its documentation for any purpose and without fee is hereby granted, provided that the above copyright notice appear in all copies and that both that copyright notice and this permission notice appear in supporting documentation. This software is provided "as is" without express or implied warranty.
>  

[Mark Petrovic](https://github.com/ae6rt) who provided valuable help in developing this code and packaging it into a stand-alone JAR
  
