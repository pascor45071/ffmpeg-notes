ffmpeg notes
================
2016-03-08



src: https://www.youtube.com/watch?v=BiMP_hN8f6s

[ffmpeg pdf slides, courtesy of the original author](https://github.com/lingtalfi/ffmpeg-notes/blob/master/ffmpeg_presentation.pdf)



Terminology
-------------

Containers are demuxed into Streams
Streams are decoded to frames
Frames are filtered
Frames are encoded back in Streams
Streams are muxed back in Containers


Container is a file format:
- streams are within containers
- examples: mp4, webm, avi.

Streams are media components
- video (h264, mpeg-2, wmv, vp8)
- audio (mp3, aac, wma)
- subtitles

Streams are formatted/stored/encoded with codec
- generally, any codec/stream can be within a container, but numerous exceptions


GOP (group of pictures)
- full frame vs differential frame
- lots of video doesn't change significantly from frame to frame



Extract mp3 from video
---------------------------

```bash 
ffmpeg -i imagine.mp4 imagine.mp3
```
- -i => next token is input
- last token is output
- audio transcoded from AAC to MP3



Get info from mp4
--------------------

```bash 
ffprobe -i imagine.mp4
```



Copy audio from video
------------------------

```bash 
ffmpeg -i imagine.mp4 -vn -c:a copy imagine.aac
```

- -vn => video none
- -c:a copy => copy audio in its input codec  (c means codec)
- VERY FAST: bitwise copy



Reduce audio bitrate
------------------------

```bash 
ffmpeg -i imagine.aac -ab 96k imagine96.aac
ffmpeg -i test.aac -ab 96k -strict -2 test96.aac
```


- -ab 96k => audio, bitrate 96 kbps (kilo bits per seconds, KB: kilobytes, kbi: use 1024 octets per kilo instead of 1000)
- requires re-encoding of audio (much slower than -c:a copy)
- output file name extension determines codec
- bit depth (vertical), possible amplitude levels
- sampling rate (hozitontal), time between samples
- bit rate (combination vertical + horizontal), bit/s
- when changing bitrate, sampling rate does not change, effectively changing bit-depth


Copy Clip 
------------

```bash 
ffmpeg -i blink_lv.mp4 -ss 11:38 -t 02:00 -c copy i_miss_you_lv.mp4
```

- -ss 13:38 => skip 13m38s into input
- -t 02:00 => duration, process input for 2 min
- -c copy => codec copy
	- shortcut for -c:v copy -c:a copy -s:a copy
	- -c is short for -codec
- notice video is messed up at start of output clip
	- because of key frames - GOP - to be discussed later



Video Snapshots
------------------

```bash 
ffmpeg -i blink_lv.mp4 -r .01 blink_lv_%03d.png
```

- -r .01 => process input at rate of .01 Hz 
	- 1 frame every 100 seconds
	- use -r 1 to make 1 frame every seconds
	- use -r .33 to make 1 frame every 3 seconds
- blink_lv_%03d.png
	- %03d => string format for each image output

On my mac, it took 230 seconds (3 minutes and 50 seconds) to create snapshots for a regular 
878Mo mp4 (duration: 39min26s),	with a rate of 1 image every 3 seconds (-r 0.33) (783 snapshots created).	



1080 to 720
---------------

```bash 
ffmpeg -i michel_soccer1.MOV -vf scale=-1:720 michel_soccer1_720.MOV
ffmpeg -i michel_soccer1.MOV -vf scale=-1:720 -strict -2 michel_soccer1_720.MOV
```

- -vf scale=-1:720 
	- v: video
	- f: filter
	- scale: scale filter, change dimensions
	- -1: width, retain width/height ratio from source
	- 720: height
- re-encoding



Animated GIF
----------------

```bash 
ffmpeg -i michael_soccer1 720.MOV -r 2 michael_soccer.gif
```

- -r 2 => take frames at 2Hz (every 0.5 seconds)



Merge Audio and Video
-------------------------

```bash 
ffmpeg -i revolusion.mp3 -i michael_soccer1_720.MOV -c:v copy -c:a copy -shortest soccer_merged.mp4
```

- -i revolusion.mp3 => audio we want
- -i michael_soccer1_720.MOV => video we want
- -shortest => end endcoding after shortest clip
- default stream mappings
	- how was audio chosen ? revolusion.mp3, michael_soccer, both?
	- -> audio with highest sampling rate (44100Hz, 48000Hz, ...) is chosen by default
	- -> if there is a tie, then it takes the audio that was first on the command line
	- -> you can override defaults with the map option


Concatenate Videos
---------------------

```bash 
ffmpeg -f concat -i file_list.txt -c copy michael_soccer_concatenated.MOV
```

- -f concat => force format concatenate format
- -i file_list.txt => file names to concatenate in file_list.txt
- -c copy => use same audio/video codecs as source
- bitwise copy

(content of file_list.txt)
```
file 'michael_soccer1.MOV'
file 'michael_soccer2.MOV'
file 'michael_soccer3.MOV'
```


Note: on my mac, this doesn't work well: I tested 2 files concatenation, and audio streams
were concatenated correctly, but the second video stream was not synced with the audio content.
I got many of the following error, which I didn't resolve:

Non-monotonous DTS in output stream 0:0; previous: 889200, current: 601600; changing to 889201. This may result in incorrect timestamps in the output file


Tips
---------------------

- -loglevel panic: make ffmpeg silent
- -y => always overwrite output w/o asking
- -n => never overwrite output






	