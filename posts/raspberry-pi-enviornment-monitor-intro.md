Title: Raspberry Pi Environment Monitor - Intro
Category: RaspberryPi

After seeing this [Raspberry Pi Thermostat Controller](http://hackaday.com/2014/11/26/custom-raspberry-pi-thermostat-controller/) on Hackaday,
I started scheming. I've had issues with my thermostat being a POS, and since it's an apartment, I really don't want to replace the thermostat
with a new one. I also had an [Adafrut 2.2" TFT](http://www.adafruit.com/products/797) laying around, looking for a use.

The first step was getting the display hooked up. Luckily, someone else has done the hard work with the [fbtft kernel module](https://github.com/notro/fbtft/wiki).
A quick breadboard later (and realizing that the SDA pin on my Pi breakout board was the I2C clock pin, and not the SPI clock), I had this:

![Some rats nest on a breadboard]({filename}media\pi-env-mon-intro\breadboard1-sm.jpg)

Which my phone camera really does not like taking a picture of that display in the low light of my office. After some fiddling with the rotation
of the display to get something natural for how I plan on looking at it. Next up was deciding what I wanted it to do:

*   Grab the local weather conditions.

    I wanted something to get the current temperature, humidity, pressure and other statuses, with an easy to use API,
    update the results periodically, and shove them into a data store. These should also be displayed on the LCD,
    perhaps with a graph (and some buttons to cycle through graphs)
 
*   Read sensor data
    
    I have sensors on order for temperature and relative humidity, along with a pressure sensor. I want to be able to
    read these, and shove them into some sort of data store. The temperature sensor will also feed into the thermostat
    control. I'd also like to expose this data over a web interface. This should also be shown on the LCD.
    
*   Control my thermostat
    
    Take the temperature data and use that to control relays hooked up to my thermostat, like in the hackaday linked project.
    Also have the status and controls for this over the web.
    
*   Support for remote sensors

    I also have some ESP8266 boards on order, and I would like one of these in my bedroom so I don't freeze my ass off when it
    gets cold, since my bedroom is the furthest room from the thermostat. I would like these to post sensor data to the main
    unit over some sort of RESTful interface.
    
Since everything but the first relies on parts I don't currently have, I'll concentrate on grabbing the local weather data, logging
that to a store, and drawing it to the LCD. I can also wire up a basic web interface once that is done.

So, on the software side of things, I've decided on the following:

*   Python
*   PyGame for drawing to the frame buffer
*   Twisted for handing HTTP requests (both incoming and outgoing), along with message passing and timed tasks
*   Redis for a data store

The various bits will talk amongst themselves using Twisted's producers and consumers. 

For the weather reporting, I've decided on [forecast.io](https://developer.forecast.io/). It has a rather simple JSON interface,
and allows 1000 API calls free per day, which considering I'm planning on updating the outside weather once every 5 minutes, that
works out to less than 300 requests per day, so I'll be well and good using their API.

Next up, getting the weather data and doing stuff with it.