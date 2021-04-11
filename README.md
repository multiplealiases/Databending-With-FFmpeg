# Databending with FFmpeg: (Ab)using Lossy Audio Compression to Databend Images

## Required Software
(These are not strictly needed to achieve the same result; this is just what I know works. Feel free to port this piece to other OSes or automate/optimize it)
 - Windows 7 or newer
 - [Irfanview](https://www.irfanview.com/)
 - [FFmpeg](https://ffmpeg.org/)
 - [VLC](https://www.videolan.org/) (optional, but used for demonstration purposes)

## Required Reading
If you wish to follow along, please read and understand this article first if you haven't already:

[Databending with Audacity: What I do Differently/Required Reading](https://github.com/multiplealiases/Databending-In-Audacity-Required-Reading/blob/main/README.md) by me

It covers my particular methodology for databending, and how it's different from the one shown in the first article. It's centered on Audacity as the databending method, but two of the fundamental ideas there (specifically about using RAW, and planar RAW at that) will be used throughout.

I'll also assume that you at least vaguely understand what FFmpeg is, and have a copy on your system somewhere. A small amount of command-line experience is needed; at least know how to change directories.

## An optional, but recommended tip: Add the FFmpeg directory to PATH temporarily.

In my last article, [Databending with Audacity: FFmpeg as an Intermediary for Images](https://github.com/multiplealiases/Databending-Audacity-FFmpeg/blob/main/README.md), I spoke of running FFmpeg right where you extracted it to. This is actually an awful idea for organization, and it means that you'll have to pile up everything you want to databend into one folder. Not ideal.

What I'd suggest instead is to add the FFmpeg folder into your current command-line session's PATH. This prevents the conflicts that may happen if you set it permanently (I don't know what your system is like, so that could break something), but the downside is that it only lasts for that one session. You close that Command Prompt window (henceforth referred to as `cmd`), it forgets the temporary PATH, and you'll have to set it again. Here's the command you need to run in a `cmd` window:

~~~
set PATH=%PATH%;<Directory where FFmpeg.exe resides>
~~~

In my case, my FFmpeg directory is ``D:\ffmpeg-2021-03-31-git-61ea0e3191-full_build\bin``, so mine would look like

~~~
set PATH=%PATH%;D:\ffmpeg-2021-03-31-git-61ea0e3191-full_build\bin
~~~

Protip: go to your FFmpeg folder in Explorer, and click on the folder icon in the address bar, like so:

![Selecting the address bar](https://i.postimg.cc/9F6wrFX6/selecting-the-address-bar.png)

Then do Ctrl+C to copy the address to paste into `cmd`. Much faster than typing that frankly *intimidating* mess of alphanumeric characters. It's like the Glitch Gremlin has a full-time job generating these names; it's perfectly normal in context.

Once that `set` business is done, try to run ffmpeg, making sure that you're outside of its folder. If you see something like:

~~~
ffmpeg version 2021-03-31-git-61ea0e3191-full_build-www.gyan.dev Copyright (c) 2000-2021 the FFmpeg developers
 built with gcc 10.2.0 (Rev6, Built by MSYS2 project)
configuration: --enable-gpl --enable-version3 --enable-static --disable-w32threads --disable-autodetect --enable-fontconfig --enable-iconv --enable-gnutls --enable-libxml2 --enable-gmp --enable-lzma --enable-libsnappy --enable-zlib --enable-librist --enable-libsrt --enable-libssh --enable-libzmq --enable-avisynth --enable-libbluray --enable-libcaca --enable-sdl2 --enable-libdav1d --enable-libzvbi --enable-librav1e --enable-libsvtav1 --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxvid --enable-libaom --enable-libopenjpeg --enable-libvpx --enable-libass --enable-frei0r --enable-libfreetype --enable-libfribidi --enable-libvidstab --enable-libvmaf --enable-libzimg --enable-amf --enable-cuda-llvm --enable-cuvid --enable-ffnvcodec --enable-nvdec --enable-nvenc --enable-d3d11va --enable-dxva2 --enable-libmfx --enable-libglslang --enable-vulkan --enable-opencl --enable-libcdio --enable-libgme --enable-libmodplug --enable-libopenmpt --enable-libopencore-amrwb --enable-libmp3lame --enable-libshine --enable-libtheora --enable-libtwolame --enable-libvo-amrwbenc --enable-libilbc --enable-libgsm --enable-libopencore-amrnb --enable-libopus --enable-libspeex --enable-libvorbis --enable-ladspa --enable-libbs2b --enable-libflite --enable-libmysofa --enable-librubberband --enable-libsoxr --enable-chromaprint
  libavutil      56. 72.100 / 56. 72.100
  libavcodec     58.135.100 / 58.135.100
  libavformat    58. 77.100 / 58. 77.100
  libavdevice    58. 14.100 / 58. 14.100
  libavfilter     7.111.100 /  7.111.100
  libswscale      5. 10.100 /  5. 10.100
  libswresample   3. 10.100 /  3. 10.100
  libpostproc    55. 10.100 / 55. 10.100
Hyper fast Audio and Video encoder
usage: ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...

Use -h to get full help or, even better, run 'man ffmpeg'
~~~
then you know it's worked. If you instead see:

~~~
'ffmpeg' is not recognized as an internal or external command,
operable program or batch file.
~~~

then you need to recheck your first command. Once you have a working `set` command, copy the entire command into a text file somewhere to run again later if you do end up having to close that `cmd` window. 

## Getting Down to Business
Or rather, some theory. Technically speaking, when you're working with uncompressed images, there's actually no difference between it and audio. It's all data anyway. Now, if I tried doing this the normal way, FFmpeg would not let you do any of this. The way to get around  this is to tell FFmpeg that your raw image data is audio, as I'll demonstrate a bit later. Then, you tell it to decode that "audio" back to its original format as if it was audio. We know full well that it's not audio, but keeping FFmpeg in the dark is how this trickery can be done in the first place.

Once you've set your PATH (assuming you didn't skip the previous step; adjust accordingly if you did), convert your images into planar RAW with Irfanview, saving with a short filename without spaces. Oh, and take note of the resolution of your image. Then go into your `cmd` window where you've set the PATH, and navigate to the folder with the images you're going to be working with. I'll be using this picture to demonstrate, naming it "cityscape.raw":

![landscape photography of city skyline at night](https://i.postimg.cc/mRKB23JH/chuttersnap-JH0w-Ceg-Jsr-Q-unsplash.jpg)

[Photo](https://unsplash.com/photos/JH0wCegJsrQ) by [CHUTTERSNAP](https://unsplash.com/@chuttersnap) on [Unsplash](https://unsplash.com)

Let's say we want to convert the raw data into MP3 at some terrible bitrate. That would go:
~~~
ffmpeg -f u8 -ar 48000 -ac 1  -i cityscape.raw -c:a libmp3lame -b:a 32k -ac 1 cityscape-mp3-32k.mkv
~~~
and you should see something along the lines of:

~~~
ffmpeg version 2021-03-31-git-61ea0e3191-full_build-www.gyan.dev Copyright (c) 2000-2021 the FFmpeg developers
  built with gcc 10.2.0 (Rev6, Built by MSYS2 project)
  configuration: --enable-gpl --enable-version3 --enable-static --disable-w32threads --disable-autodetect --enable-fontconfig --enable-iconv --enable-gnutls --enable-libxml2 --enable-gmp --enable-lzma --enable-libsnappy --enable-zlib --enable-librist --enable-libsrt --enable-libssh --enable-libzmq --enable-avisynth --enable-libbluray --enable-libcaca --enable-sdl2 --enable-libdav1d --enable-libzvbi --enable-librav1e --enable-libsvtav1 --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxvid --enable-libaom --enable-libopenjpeg --enable-libvpx --enable-libass --enable-frei0r --enable-libfreetype --enable-libfribidi --enable-libvidstab --enable-libvmaf --enable-libzimg --enable-amf --enable-cuda-llvm --enable-cuvid --enable-ffnvcodec --enable-nvdec --enable-nvenc --enable-d3d11va --enable-dxva2 --enable-libmfx --enable-libglslang --enable-vulkan --enable-opencl --enable-libcdio --enable-libgme --enable-libmodplug --enable-libopenmpt --enable-libopencore-amrwb --enable-libmp3lame --enable-libshine --enable-libtheora --enable-libtwolame --enable-libvo-amrwbenc --enable-libilbc --enable-libgsm --enable-libopencore-amrnb --enable-libopus --enable-libspeex --enable-libvorbis --enable-ladspa --enable-libbs2b --enable-libflite --enable-libmysofa --enable-librubberband --enable-libsoxr --enable-chromaprint
  libavutil      56. 72.100 / 56. 72.100
  libavcodec     58.135.100 / 58.135.100
  libavformat    58. 77.100 / 58. 77.100
  libavdevice    58. 14.100 / 58. 14.100
  libavfilter     7.111.100 /  7.111.100
  libswscale      5. 10.100 /  5. 10.100
  libswresample   3. 10.100 /  3. 10.100
  libpostproc    55. 10.100 / 55. 10.100
[u8 @ 000002816bb8ea00] Estimating duration from bitrate, this may be inaccurate
Guessed Channel Layout for Input Stream #0.0 : mono
Input #0, u8, from 'cityscape.raw':
  Duration: 00:02:33.72, bitrate: 384 kb/s
  Stream #0:0: Audio: pcm_u8, 48000 Hz, mono, u8, 384 kb/s
 Stream mapping:`
  Stream #0:0 -> #0:0 (pcm_u8 (native) -> mp3 (libmp3lame))
Press [q] to stop, [?] for help
Output #0, matroska, to 'cityscape-mp3-32k.mkv':
  Metadata:`
    encoder         : Lavf58.77.100`
  Stream #0:0: Audio: mp3 (U[0][0][0] / 0x0055), 48000 Hz, mono, s16p, 32 kb/s
    Metadata:`
      encoder         : Lavc58.135.100 libmp3lame
size=     640kB time=00:02:33.69 bitrate=  34.1kbits/s speed= 134x
video:0kB audio:601kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 6.528222%
~~~


### A breakdown of the command I just wrote
~~~
ffmpeg
~~~
Self-explanatory; runs ffmpeg.

#### Input section
~~~
-f u8
~~~
Specifies a format of unsigned 8-bit PCM audio. 

~~~
-ar 48000
~~~
Specifies an audio sample rate of 48000 Hz.

~~~
-ac 1
~~~
Tells FFmpeg to read the raw data as mono audio.

~~~
-i cityscape.raw
~~~
Specifies an input file called "cityscape.raw".

#### Output section
~~~
-c:a libmp3lame
~~~
Specifies the libmp3lame encoder, otherwise known as just LAME. If you've had to install this thing called "LAME" in older versions of Audacity in order to save audio as MP3, that is exactly what this is.
~~~
-b:a 32k
~~~
Sets a bitrate of 32 kbps (though considering the very noisy nature of raw image data, this is mostly a suggestion).

~~~
-ac 1
~~~
Tells FFmpeg to generate mono audio on the output side.

~~~
cityscape-mp3-32k.mkv
~~~
Generates an output file called "cityscape-mp3-32k.mkv", mkv indicating the use of a Matroska container format.

#### Why MKV, not MP3?
MKV is a universal container format. A container format lets you combine multiple streams (like audio and video) into one file, but they're not all are created equal. MP4, for instance, allows only a limited set of audio and video formats. MKV is different, and lets you put anything and everything into it. If you can think it, it's probably supported. It's easier to just put .mkv at the end, instead of having to remember which formats go with which exact container. This breaks compatibility with a lot of players, but VLC and FFmpeg shouldn't have problems with it.

## Converting the audio back into raw data

You can actually go and play the audio that you just made. Just open it (in VLC, preferably), and you'll hear low-quality screams of the damned. Perfectly normal by databending standards.

Alright, let's convert it back into raw data. Here:

~~~
ffmpeg -i cityscape-mp3-32k.mkv -f u8 -ar 48000 -ac 1 cityscape-mp3-32k.raw
~~~

###  Breaking it down
~~~
-i cityscape-mp3-32k.mkv
~~~
Specifies an input file called "cityscape-mp3.32k.mkv". Since the instructions needed to decode the file are already inside it, it's unnecessary to set anything further.

#### Output side
~~~
-f u8
~~~
Specifies an audio format of unsigned 8-bit PCM. 

~~~
-ar 48000
~~~
Specifies an audio sample rate of 48000 Hz.

~~~
-ac 1
~~~
Tells FFmpeg to read the raw data as mono audio.

~~~
cityscape-mp3-32k.raw
~~~
Generates a file called "cityscape-mp3-32k.raw" as output.

## Reading the output file

Let's open it in Irfanview.
![enter image description here](https://i.postimg.cc/BqqHY8xh/cityscape-mp3-32k-misaligned.png)

And it's been shifted by some amount. Reopen it with different file header offsets until you end up with a properly-aligned picture.

![enter image description here](https://i.postimg.cc/hKKXgkfv/cityscape-mp3-32k.png)

It's not too drastic here, but you can see that's it's had an effect on the image. As a general rule, lower bitrates mean increased distortion. It's not extremely obvious here, but other encoders, formats, and bitrates can yield different results.

## Testing out different encoders at various bitrates
### libshine (MP3 encoder)
#### 32 kbps

![enter image description here](https://i.postimg.cc/JRFjNSZp/cityscape-libshine-mp3-32k.jpg)

#### 64 kbps

![enter image description here](https://i.postimg.cc/ysvmdDd4/cityscape-libshine-mp3-64k.jpg)

#### 128 kbps

![enter image description here](https://i.postimg.cc/VYbnD4Fm/cityscape-libshine-mp3-128k.jpg)

(In case you're wondering, no, this never recovers. It always produces this odd coloration effect regardless of the bitrate you pick)

### libopus (Opus encoder)
#### 6 kbps

![enter image description here](https://i.postimg.cc/0PffwtV4/cityscape-libopus-6k.jpg)

#### 8 kbps

![enter image description here](https://i.postimg.cc/Vsw-qSMn1/cityscape-libopus-8k.jpg)

#### 16 kbps

![enter image description here](https://i.postimg.cc/jxP6dvWZ/cityscape-libopus-16k.jpg)

#### 32 kbps

![enter image description here](https://i.postimg.cc/Rvhcnv63/cityscape-libopus-32k.jpg)

#### 64 kbps
![enter image description here](https://i.postimg.cc/yBCXrq55/cityscape-libopus-64k.jpg)

### opus (experimental Opus encoder in FFmpeg)
#### 6 kbps

![enter image description here](https://i.postimg.cc/0PffwtV4/cityscape-libopus-6k.jpg)

#### 8 kbps

![enter image description here](https://i.postimg.cc/Vsw-qSMn1/cityscape-libopus-8k.jpg)

#### 16 kbps

![enter image description here](https://i.postimg.cc/jxP6dvWZ/cityscape-libopus-16k.jpg)

#### 32 kbps

![enter image description here](https://i.postimg.cc/Rvhcnv63/cityscape-libopus-32k.jpg)

#### 64 kbps

![enter image description here](https://i.postimg.cc/yBCXrq55/cityscape-libopus-64k.jpg)

### libvorbis (Vorbis encoder)
#### 32 kbps

![enter image description here](https://i.postimg.cc/X3nBCntV/cityscape-libvorbis-32k.jpg)

#### 64 kbps

![enter image description here](https://i.postimg.cc/3YckVmNR/cityscape-libvorbis-64k.jpg)

#### 128 kbps

![enter image description here](https://i.postimg.cc/bpJGm3zW/cityscape-libvorbis-128k.jpg)

### aac (FFmpeg's native AAC encoder)
#### 6 kbps

![enter image description here](https://i.postimg.cc/NsHD2Bn8/cityscape-aac-6k.jpg)

#### 8 kbps

![enter image description here](https://i.postimg.cc/kC0f3RVT/cityscape-aac-8k.jpg)

#### 16 kbps

![enter image description here](https://i.postimg.cc/VsPRj2yx/cityscape-aac-16k.jpg)

#### 32 kbps

![enter image description here](https://i.postimg.cc/9cQBtGSQ/cityscape-aac-32k.jpg)

#### 64 kbps

![enter image description here](https://i.postimg.cc/tyFDXRWp/cityscape-aac-64k.jpg)

#### 128 kbps

![enter image description here](https://i.postimg.cc/fDZCk4fX/cityscape-aac-128k.jpg)

### libtwolame (MP2 encoder)
#### 32 kbps

![enter image description here](https://i.postimg.cc/wgfNznf9/cityscape-libtwolame-32k.jpg)

#### 64 kbps

![enter image description here](https://i.postimg.cc/wHkJf1Dy/cityscape-libtwolame-64k.jpg)

#### 128 kbps

![enter image description here](https://i.postimg.cc/ZT3vrnTq/cityscape-libtwolame-128k.jpg)



### libspeex (Speex encoder)

#### 8 kbps

![enter image description here](https://i.postimg.cc/54BvM0Gr/cityscape-libspeex-8k.jpg)

#### 16 kbps

![enter image description here](https://i.postimg.cc/rcTxN1R2/cityscape-libspeex-16k.jpg)

#### 32 kbps

![enter image description here](https://i.postimg.cc/0P2SL9Ct/cityscape-libspeex-32k.jpg)

#### 40 kbps

![enter image description here](https://i.postimg.cc/66BZ6Kxp/cityscape-libspeex-40k.jpg)

### libvo-amrwbenc (Adaptive Multi-Rate Wideband encoder)

#### 6.6 kbps

![enter image description here](https://i.postimg.cc/x9Xz1ghM/cityscape-libvo-amrwbenc-6600.jpg)

#### 15.85 kbps

![enter image description here](https://i.postimg.cc/gzWX4pJH/cityscape-libvo-amrwbenc-15850.jpg)

#### 23.85 kbps

![enter image description here](https://i.postimg.cc/8kkFQbLF/cityscape-libvo-amrwbenc-23850.jpg)

## Conclusion

While I don't think it's quite as spectacular as applying Paulstretch on images, I think it's still an interesting effect to apply. There are a lot of audio codecs out there (though you'd want lossy ones, not lossless), so there's all sorts of fun you could have with that.

If there's anything you'd like to add or change, fork (and add a pull request) this repository or contact me on GitHub at [multiplealiases](https://github.com/multiplealiases).


