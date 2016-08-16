Title: Movies on Stellaris - Part 1
Category: Stellaris
Date: 2013-2-17 02:15

So, I got a [Stellaris Launchpad](http://www.ti.com/ww/en/launchpad/stellaris_head.html) the other day,
and was wondering a project to do with it. After a bit of thought, and looking at the various parts I
had sitting around, which included a [TFT LCD Display with an SD Card](http://adafruit.com/products/797),
and thinking back to a project I'd seen before, I decided for my first project, I'd see how fast I can
pull data from the SD card and write it out to the LCD (both over SPI).

The video file? Since I didn't want to write any complex decoding at this point on the board, I decided
to have a basic format that's RLE encoded. And what video would encode quite well with RLE? I'm glad you asked:

<iframe width="420" height="315" src="http://www.youtube.com/embed/9lNZ_Rnr7Jc" frameborder="0" allowfullscreen></iframe>

Simple black and white. I'm not going to worry about the audio right now.

The first step: How on earth to read the video, and then write it back out encoded? The answer? OpenCV and Python.
A half dozen lines to read the video frame by frame, smash down the color depth (not really needed right now, but
it's there in case I want to process the image further at some point), and then shove the image into a byte array.

For testing, I wrote a small C# program that reads in the files one by one, creates an image out of them, and shoves
them into a picturebox on a form. Also simple.

A sanity check: Can I take the video frames, encode them via a brain dead method, and read them back out?

![Image](/static/media/stellaris/decode001.png)

Yep. Although this is about the most brain dead method possible: Each pixel is two bytes, one indicating a run
length of 1, the other storing the value of the pixel.

Now, let's try using RLE encoding to get something out of it. The RLE variant I use works as follows:

    Use itertools.group to get byte runs in the data
    
    Each group back from itertools:
       Is the length greater than the minimum run?
          Flush the small run buffer
          Write out run length + 128
          Write out byte value for run
        else
          if small runs buffer + current run >=127?
             Write out run length
             Write out small runs buffer
          Add run to small runs buffer

Very rough psuedocode. Essentially, if a byte value > 127, the byte value - 128 is the length of the run,
with the next byte being the data byte. If it's < 127, the next n bytes are treated as literal. There's some
added logic if the run length is greater than 127 where it figure out how many 127-length runs to write out.

Overall efficiency isn't the best of course, since this is a rather simple method of compression. The largest
frame at 23011 bytes is this one:

![Image](/static/media/stellaris/largestimage.jpg)

Lots of grays and shading. Closely followed by this one at 23009 bytes:

![Image](/static/media/stellaris/largestimage2.jpg)

Once again, rather expected worst case. I've attempted some dithering in the pipeline to help. Overall size
for 6568 frames is about 40MB, reduced to a resolution that will fit on the LCD display I have. The video runs 
at 30fps, I plan on reducing that to 15fps at first, which would reduce the size to roughly 20MB. So, my target 
transfer rate off the SD card over the SPI bus is about 93KBps. Some preliminary research puts this in the 
"very possible" range. 

I plan on joining together the loose files in a rather simple format of (file length)(file data)(file length)(file data)
, etc, with the final length being 0. This gives roughly 6.5kb of overhead, since I can cram the file lengths into 16 bit
ints.

Next up, poking at the chip itself.