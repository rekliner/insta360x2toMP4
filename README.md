# Insta 360 x2 fisheye to google VR180 Mapping (directly from files)

Many '360°' camera's, such as the [insta 360 x2](http://insta360.com/), consist of 2
fish-eye camera's. I've added a couple of presets to raboof's code to facilitate taking files directly from the SD card to Youtube.

## Why DIY?

Those camera's typically come with desktop software or apps
to manipulate the images and for example share to facebook.

It's fun to explore doing this without relying on the official software. 

Also, the official insta360 software is cumbersome, filled with ads, and terrible for long videos...though it does have nice stitching options.

## Storage

The x2 stores its photos and videos on its memory card in INSP and INSV
format, easily accessible via USB storage without even removing the card.
It saves each fisheye lens as a separate file and also splits the files at the 30 minute mark which makes dealing with long videos excruciating.  
Note that if you're looking at the camera screen the front fisheye is the .insp file that has "_10_" after the date-time stamp.. like so: VID\_20211204\_182041<b>\_10\_</b>007.insp

## Projection conversion

Those images and video's show the 'double fish-eye' nature of the device.
Services like Facebook, however require 360° imagery to be mapped using the Equirectangular Projection. 
Youtube is able to handle the fisheye format for VR180 with the right metadata, but one of the lenses needs to be doubled to emulate VR.   
Note that the Insta 360 X2's fisheye images are rotated 90 degrees when held upright...for no useful reason I can see.  

This is achieved with `ffmpeg` using 2 'mapping files' for your image type.  

I haven't yet worked on 360 degree exporting since I only need 180 for this project.  The PC and phone apps have very elaborate stitching and file merging functions that don't have open-source equivalents...it makes sense to use the insta apps for those features.  Since I only need 1 lens I bypass the app and only re-encode once.

I also haven't worked with its photos yet.  The INSP format appears to be a jpg with both lenses merged into equirectangular 360 projection already.

## Mapping generation

The original coder found `projection.c` by Floris Sluiter which could generate mapping files for single-fisheye sources so single equirectangular outputs.  
They modified it to support double-fisheye ins and outs.  
I've added mappings for a single fisheye from the X2 to output a double fisheye video suitable for Youtube VR180 with the right metadata.

Compile the generator code:

    gcc -o projection projection.c -lm

Create mapping files for videos:

    ./projection -x xmap_x2f_video.pgm -y ymap_x2f_video.pgm -h 2880 -w 2880 -r 2880 -c 5760 -m x2_1f

## Usage

Once you have created (or downloaded) the mapping files, you can use them diretly with ffmpeg.  

    ffmpeg -i in_file.insp -i xmap_dokicam_video.pgm -i ymap_dokicam_video.pgm -filter_complex remap out.mp4 

To force in VR180 metadata use Vargol's fork of Kodabb's fork of Google's [Spatial Metadata Injector](https://github.com/Vargol/spatial-media).  which I'll include as a subproject here


## Tying it all together: *x2upload.py*
I've written a simple python gui to select files to export (it will append/merge them in reverse order of their selection the way windows file selection normally works). It remaps them and merges them with ffmpeg.  It adds the metadata with the injector to the single output file.  

## Youtube integration for funsies
If you're really feeling technical and have a domain and a few days to deal with google's ridiculous oauth API application verification process....  
You can set up verified youtube oauth2 API credentials and it will upload the resulting file directly to your youtube channel.  

##  Conclusion
Re-encoding hour-long VR videos is painfully slow using any process, so this really helps to have it be one operation that can be left running without interaction.
Let me know if you have suggestions.  Example videos at [this youtube channel](https://www.youtube.com/channel/UCCXKvgol2YfeDmCFNm9CYbg/videos )
