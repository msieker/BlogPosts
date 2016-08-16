Title: Raspberry Pi Environment Monitor - New Sensors and a Web API
Category: RaspberryPi

Now that my shipments from China, Digikey, Adafruit and other parts of the world have arrived,
I should probably write an update on this project.

First off, I have some [MCP9808 temperature sensors](https://www.adafruit.com/product/1782). These
have greatly improved resolution over the DHT11 I was using before, as shown by this chart:

![A very colorful chart with wiggly lines.]({filename}media\pi-env-mon-intro\temphistory.png)

The left bit of data is when the DHT was used for both temperature and humidity. The flat line is
when the Pi was powered off for a bit, and then it resumes with the DHT still used for humidity,
but the MCP used for temperature. It is now very clear when the furnace turns on and off. You could
make it out in the previous data, but with much greater resolution.

To interface with the new sensor, which is I2C based, I once again turned to Adafruit with their
[Python Library for the MCP9808](https://github.com/adafruit/Adafruit_Python_MCP9808). Integration
basically amounted to just putting a call to it when I read my sensor:

<script src="https://gist.github.com/msieker/d43720174a193a3f3340.js"></script>

Since the DHT sensor is the slower, I let that drive the rate of sampling. Then I disregard the
temperature information from the DHT, and just read the MCP sensor, and log that. The same 10
second rolling average is there to smooth the data.

Now that I have this data, I need some way to get to it. I started work on implementing a somewhat
RESTful API using Twisted. There are endpoints for sensor and weather data currently, that are set
up like this:

<script src="https://gist.github.com/msieker/a96ba78a462dad3ec249.js"></script>

I can pass how much data I on the query string, compute the number of minutes from that, and pass
that on to my Redis wrapper. For easier consumption, I add a string version of the date and time
to the outgoing data. To make this data easier to consume, I made the helper function `renderCsvOrJson`:

<script src="https://gist.github.com/msieker/f81693cbc6241c9e50dd.js"></script>

If `application/json` is not explicitly in the `Accept` header, I return CSV data with instructions
that it is an attachment, so the browser downloads a CSV file. This lets me, for now, easily make
my charts by just visiting the endpoint and waiting for the CSV to generate and download.

On the topic of generating data sets, once I implemented my API, I noticed that it took excessive
amounts of time to generate the data. This was traced down to the N+1 round trips to Redis to get
the data:

<script src="https://gist.github.com/msieker/d41e5cb6c0070ab92059.js"></script>

Knowing that Redis had a Lua scripting interface, I turned to this. But then I discovered that 
Raspbian has a rather old version of Redis... from before they added Lua. This lead to the fun
process of updating my Pi to run the `Testing` branch, which took ages to install the packages.
Once this was done, I wrote my first Lua, ever. And reading the documentation, it seemed that
msgpack was the best option to pass data between Redis and Python.

<script src="https://gist.github.com/msieker/bc03b54907c615adbe6f.js"></script>

With this I can pull a weeks work of data in the time it took to pull 8 hours of data previously.

And now, and email I just got leads me to mention something else I've worked on:
![A screenshot with some stuff.]({filename}media\pi-env-mon-intro\boards.png)

I decided to play with KiCad last week, and try my hand at designing a PCB. The board itself is
a rather simple IO expansion, with the pair of PCF8574s adding pins for buttons and the relay
board, with a header for the new 320x240 LCD display that came from China. I decided to design
this board for the Raspberry Pi B+ so I had access to more IO, and better placement of the Vcc
pins for both +3.3v and +5v. I've ordered some extra-tall stacking headers for this board so I
can plug my breakout cable into this board. This will clear up a bit of the rats nest on my
breadboard, and the large amount of space taken by the LCD. The board itself is 2mm larger than
the Pi in both width and height, to give me a slightly easier time routing, and to actually give
me room for the buttons (Which, I just noticed the ground and power fills are not even across
the bottom of the board. Oh well, that's what v1.1 is for, along with moving the resistors to
the middle of the board vertically). Considering I never did PCB design before, and I was able
to make a board ready for fabrication in two days, the learning curve for Kicad wasn't that bad
though, it does have its quirks.

For fabrication, I remember reading about [Dirty PCBs](http://dirtypcbs.com) awhile back on
Hack-A-Day, and decided to use them. It is somewhat odd that they offer cheap 5x5 boards, 
which are smaller than both the Arduino shield, and the Pi. It's still the cheapest option
though. [My board is here](http://dirtypcbs.com/view.php?share=3252&accesskey=535a2f6625c960390735f8c46bda1e5c)
if for some reason you want an amateur designed Raspberry Pi IO expansion board. 