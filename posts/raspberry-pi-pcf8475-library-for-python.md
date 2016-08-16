Title: Python library for the PCF8475 on the Raspberry Pi
Category: RaspberryPi

While there was progress on my environment monitor project tonight, I'm going to write that
up later... er, today, considering it's after 4AM currently. I'm going to do a quick writeup
instead on a side project that is related.

Part of my project is interfacing with a pair of 
[PCF8475 I2C IO expander](http://www.ti.com/lit/ds/symlink/pcf8574.pdf) chips to control
a relay board, and read buttons. To this end, I've created a small Python class to interface
with these chips, [available on Github](https://github.com/msieker/python_pcf8475). It supports
basic reading and writing to the chips, along with being able to trigger a callback when the
`/INT` line is triggered. Usage is quite simple:

<script src="https://gist.github.com/msieker/889ebe603efce1f3efd0.js"></script>

Eventually I'm going to add the ability to toggle a specific pin, while maintaining the state
of the other pins.