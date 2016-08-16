Title: Raspberry Pi Environment Monitor - Displaying simple stuff
Category: RaspberryPi

[Now that I have data]({filename}raspberry-pi-enviornment-monitor-forecast.io-integration.md), I
need to be able to see that data. Time to play with PyGame and the frame buffer.

After much reading of the PyGame docs, I noticed the oddity that text is always drawn on its
own surface. Not a major thing, but different from most things I've dealt with in the past.
This also involved a long wait to install the Droid font family. Droid Sans Mono has been
my favorite monospace font for awhile, and looks good at small font sizes, which is what I'm
dealing with here on this 220x176 screen.

<script src="https://gist.github.com/msieker/2f85e4af22cbc262e51b.js"></script>

The frame buffer setup code is basically ripped from the 
[Pointing Pygame to the Framebuffer](https://learn.adafruit.com/pi-video-output-using-pygame/pointing-pygame-to-the-framebuffer)
sample on Adafruit, with a bit of added code to specify the frame buffer, set up my drawing
surface and hide the mouse cursor.

Currently I have the display refreshing about twice a second, mainly to ensure I don't wait
too long drawing the clock displayed, otherwise that looks kind of strange. With that refresh
rate, the SPI driver uses about 6% of the CPU on the Pi, with Python using another 5%. Screen
drawing should be my highest continuous task, as most other tasks I have planned for this to
do range from one every few seconds to once every few minutes. Though, these numbers will
probably go up a bit once I get my new display that's a whopping 320x240.

![A bad photo of some text on a screen]({filename}media\pi-env-mon-intro\simpledisplay.jpg)

This display looks much better in real life. I should get an overhead lamp on my work table.

I need to work out a good way to make it easier to set up what goes on the screen where than
hard-coding all of it. But that can wait until I have the display with the size I'll be using.

The display does not yet update the weather as the weather updater pushes the updated conditions
out, since I do not have the subscribe parts of the Redis wrapper implemented. That will probably
be next on my list of things to do. It does, however, use a new method on the Redis wrapper
to do the initial load of the data, `GetLatestTimeSeriesMember`:

<script src="https://gist.github.com/msieker/e9954fad7464e72cd7f5.js"></script>

This call will remain even after I have the push updates implemented, since the latest update
might not be available when the display first draws.