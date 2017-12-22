# Time-Lapse assembling

This repo contains scripts used to compile Time-lapse movies using ffmpeg.
The ffmpeg utility is controlled via the ffmpeg-python wrapper.


# Codec

For near-universal compatibility the H.264 codec is used. The following
section describe some of the choices for specific configuration options.

See the ffmpeg wiki for additional information:
https://trac.ffmpeg.org/wiki/Encode/H.264


## Constant Rate Factor

Use `-crf 22` to set a constant rate factor, which means the overall
quality of the movie should be constant. So bitrate varies to ensure
this. Higher value is slower quality. The quality with 22 seems
reasonable. Check if it streams nicely to users on slower bandwidth..
otherwise a high crf (lower quality) might be needed.


## Preset

Use `-preset slower` to improve the compression ratio for the selected
quality (crf), without taking too much time. The slower preset is still
fast enough for me.


## Faststart

Set `-movflags +faststart` to allow the movie to quickly start playing,
while it is still loading.


## Quicktime support

The codec defaults to YUV 444, which is not supported by Quicktime.
So add `-pix_fmt yuv420p` to fix Quicktime support.


# Input

## Select input frames

Use frames as input by giving a glob pattern which matches the desired images.
Usually these will be tiff images so use `-pattern_type glob -i "*.tiff"`.


## Framerate

When using image sequences as input the framerate of the desired output
should be given using `-framerate 30`.


# Filters

Commonly used filters:

- Scale https://ffmpeg.org/ffmpeg-filters.html#scale
- Crop https://ffmpeg.org/ffmpeg-filters.html#crop
- Deflicker https://ffmpeg.org/ffmpeg-filters.html#toc-deflicker

Goal; scale and crop videos to fit the desired final resolution
Perhaps use 


## Scale video

Add scaling to ensure it fits in given dimensions. Negative values for width
or height make the encoder figure out the size by itself, keeping the aspect ratio
of the source. The integer of the negative value, i.e. 4 for -4, means that the size
should be devisble by that value. TODO: does it just scale/squish the video or crop?

    -vf scale=1920:-2
    -vf scale=960:540:aspect..


# Convert movie

Using ffmpeg-python:

    import ffmpeg
    (ffmpeg
        .input('APL*.tiff', pattern_type='glob', framerate=24)
        # .filter_('scale', size='qhd', force_original_aspect_ratio='increase')
        .filter_('scale', size='hd1080', force_original_aspect_ratio='increase')
        .filter_('deflicker', mode='pm', size=10)
        .output('SD.mp4', crf=20, preset='slower', movflags='faststart', pix_fmt='yuv420p')
        .run()
    )


# Old ffmpeg CLI examples

    ffmpeg -i $1 \
        -c:a copy -c:v libx264 -preset slower -crf 22 -pix_fmt yuv420p $1_HD.mp4 \
        -c:a copy -c:v libx264 -preset slower -crf 22 -pix_fmt yuv420p -vf scale=960:-1 $1_SD.mp4 \

    ffmpeg -framerate 15 -pattern_type glob -i "*.tiff" \
        -c:a copy -c:v libx264 -preset slower -framerate 15 -crf 22 -pix_fmt yuv420p S60_050504_1_HD.mp4 \
        -c:a copy -c:v libx264 -preset slower -framerate 15 -crf 22 -pix_fmt yuv420p -vf scale=960:-1 S60_050504_1_SD.mp4

    ffmpeg -framerate 24 -pattern_type glob -i "*.png" \
        -c:a copy -c:v libx264 -preset slower -framerate 24 -crf 22 -pix_fmt yuv420p D700_140608_2_HD.mp4
    ffmpeg -framerate 24 -pattern_type glob -i "*.png" \
        -c:a copy -c:v libx264 -preset slower -framerate 24 -crf 22 -pix_fmt yuv420p -vf scale=960:-1,crop=960:624:0:50 D700_140608_2_SD.mp4

    NAME=D500_170909_1; FPS=24; DEFLICKERSIZE=10 \
    ffmpeg -framerate $FPS -pattern_type glob -i "*.tiff" \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -vf deflicker=mode=pm,scale=960:-2 ${NAME}_SD.mp4 \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -vf deflicker=size=${DEFLICKERSIZE}:mode=pm ${NAME}_4K.mp4 \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -vf deflicker=size=${DEFLICKERSIZE}:mode=pm,scale=1920:-2 ${NAME}_HD.mp4 \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -profile:v baseline -vf deflicker=size=${DEFLICKERSIZE}:mode=pm,scale=480:-2 ${NAME}_LD.mp4

    NAME=ARN_; FPS=24; ffmpeg -framerate $FPS -pattern_type glob -i "*.tif" \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -vf scale=1920:-2 ${NAME}_HD.mp4 \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -vf scale=960:-2 ${NAME}_SD.mp4 \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -profile:v baseline -vf scale=480:-2 ${NAME}_LD.mp4

    NAME=ARM_005085_trim; FPS=24; ffmpeg -framerate $FPS -i ${NAME}.mov \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p ${NAME}_HD.mp4 \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -vf scale=960:-2 ${NAME}.mp4 \
        -c:a copy -c:v libx264 -preset slower -framerate $FPS -crf 20 -movflags faststart -pix_fmt yuv420p -profile:v baseline -vf scale=480:-2 ${NAME}_LD.mp4