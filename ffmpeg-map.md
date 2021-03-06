ffmpeg map
===============
2016-03-08




From man ffmpeg:

By default, ffmpeg includes only one stream of each type (video, audio, subtitle) present in the input files and adds them to each output file.
It picks the "best" of each based upon the following criteria: for video, it is the stream with the highest resolution, for audio, it is
the stream with the most channels, for subtitles, it is the first subtitle stream. 
In the case where several streams of the same type rate equally, the stream with the lowest index is chosen.

You can disable some of those defaults by using the "-vn/-an/-sn" options.
For full manual control, use the "-map" option, which disables the defaults just described.





Now my explanations
-----------------------

Short version
-----------------

```bash
ffmpeg -i tears_of_steel.mkv -map 0:0 -map 0:2 -map 0:1 -map 0:4 -map 0:3 -c copy tears_of_steel-v2.mkv 
```

is mapped like this:

```bash
Stream mapping:
Stream #0:0 -> #0:0 (copy)
Stream #0:2 -> #0:1 (copy)
Stream #0:1 -> #0:2 (copy)
Stream #0:4 -> #0:3 (copy)
Stream #0:3 -> #0:4 (copy)
```


So, the map option's definition could be written like this:

```bash
	-map INPUT_INDEX:STREAM_INDEX (=> output 0:OUTPUT_STREAM_INDEX++ )
```



Long version
----------------

When you write a ffmpeg command, you can use multiple inputs, prefixing each input with the -i option.
For instance, this is a "complex" (at least for me) command to merge audio and video (more about this command [here](https://github.com/lingtalfi/ffmpeg-notes/blob/master/ffmpeg.md#merge-audio-and-video)) 

```bash
ffmpeg -i revolusion.mp3 -i michael_soccer1_720.MOV -c:v copy -c:a copy -shortest soccer_merged.mp4
```

So in the above commands, we have two inputs:

- revolusion.mp3
- michael_soccer1_720.MOV



Now every time you write a map option on the command line, it maps what directly follows to the implicitely auto-incrementing stream in the output.
The implicitely auto-incrementing part was what I didn't understand first.
Below is the example that enlightened my understanding of the map option (french source: http://debian-facile.org/doc:media:ffmpeg):


	-map INPUT_INDEX:STREAM_INDEX 	(=> output 0:OUTPUT_STREAM_INDEX++ )

This was the example that allowed me to understand.
Imagine the following command:

```bash
ffmpeg -i tears_of_steel.mkv -map 0:0 -map 0:2 -map 0:1 -map 0:4 -map 0:3 -c copy tears_of_steel-v2.mkv 
```

And here is how ffmpeg does the mapping:

```bash
Stream mapping:
Stream #0:0 -> #0:0 (copy)
Stream #0:2 -> #0:1 (copy)
Stream #0:1 -> #0:2 (copy)
Stream #0:4 -> #0:3 (copy)
Stream #0:3 -> #0:4 (copy)
```

What happened?

In the command, we have only one input: tears_of_steel.mkv.
That's why all our maps start with #0:...

Then we write a first map option: **-map 0:0**, which refers to the first stream of our input (tears_of_steel.mkv), which might be a video stream, but that doesn't matter.

Because it's the first map option, it will be mapped to the first stream in our output file (tears_of_steel-v2.mkv).

Then we write a second map option: **-map 0:2**, which refers to the third stream of our input. Because it's the second map option, it is mapped to the second stream in our output.

And so on...




What if you have multiple inputs then?
------------------------------------------------

 I've not been that far yet, so: I don't know.









