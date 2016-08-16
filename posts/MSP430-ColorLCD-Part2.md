Date: 2012-07-15 20:00
Title: Using the Sparkfun LCD shield with the MSP430 - Part 2
Category: MSP430

First, I now have the code for this on Github: [MSP430NokiaLCD](https://github.com/msieker/MSP430NokiaLCD).
Several things have changed in this code from the the earlier posting:

1. SPI bit-banging loops are unrolled. I wish there was some way I could get rid of the if in there...
2. Some drawing functions are now in place: `LcdDrawPixel`, `LcdDrawLine`, `LcdDrawRect`. Most of these
   functions come from James Lynch's documentation mentioned in the previous post, modified to work with
   my bit-banging routines.
3. After finding the [Universal Color LCD graphics library](http://www.43oh.com/forum/viewtopic.php?t=2476#p18263)
   on the 43oh forums, I initially tried swapping out my library with this one, and seeing if I could get   
   it to work with my LCD. No dice. However, looking through it, I modified my library from 12-bit color to
   8 bit palletized color. The loss of color depth probably isn't noticeable on these screens that much. And
   there's less data to move across the wire.

I've got a bouncing square going, and I think I have it animating about as fast as I can without too much
tearing or blurring. I think it's about as best as I'm going to get out of this LCD:

<iframe width="853" height="480" src="http://www.youtube.com/embed/6w1i2O4-Nig" frameborder="0" allowfullscreen></iframe>

Here's cycling through the palette:

<iframe width="853" height="480" src="http://www.youtube.com/embed/e300Eu0kGdk" frameborder="0" allowfullscreen></iframe>

Compared to the previous 12 bit code:

<iframe width="853" height="480" src="http://www.youtube.com/embed/LS4eUujb1Gk" frameborder="0" allowfullscreen></iframe>

It's a decent improvement.

Next, I'll probably work on text rendering. I seem to recall a minimal font where each letter was made up of a handful
of pixels, but now I can't remember what it's called. I might just use Droid Sans Mono for it.