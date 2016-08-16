Title: Raspberry Pi Environment Monitor - Push Updates and Sensors
Category: RaspberryPi

[With things being put on the display]({filename}raspberry-pi-enviornment-monitor-simple-display.md),
the ability to update that display with live data is needed. Most of the infrastructure is already in
place, it just needs to be finished up and tied together. And also, my first shipment of random tat
arrived today, including my DHT11 sensors and the relay board.

The Redis wrapper gained a new member `GetDict` that loads a hash from Redis, and had `Subscribe`
implemented. `Subscribe` takes a callback method that gets passed into an instance of `RedisCallbackSubscriber`,
which inherits from txredis' RedisSubscriber.

<script src="https://gist.github.com/msieker/1f3489b2fbc60054b2f7.js"></script>

The display manager gained a callback method that gets passed into `Subscribe`:

<script src="https://gist.github.com/msieker/5505c10336674e898e17.js"></script>

Simple matter of looking at the channel, and pulling the hash out of Redis, and putting it in the right place.
And now I have a very simple current conditions display on my little LCD.

During sensor implementation, since the script needs to run as root to have access to GPIO, pygame
behaved badly. I'm not quite sure the root cause, it seems to be some interaction between pygame
and twisted, but it would get wedged to the point that SIGKILL does nothing. For now, in the interest
of making things work, I've moved pygame initialization out into my main script, and added a LoopingCall
to pump pygame messages. This at least lets me handle keyboard keys now, and the program will actually quit.
I don't like it, since I can no longer run the display manager on its own.

My current sensors are DHT11's. I got a pack of 4 cheap on Amazon. Then I'm using the 
[DHT-sensor-library](https://github.com/adafruit/DHT-sensor-library) put out by adafruit.

<script src="https://gist.github.com/msieker/0a3977c8db3689e1148f.js"></script>

Most of it is just a bunch of setup code. The only interesting bits involve `deferToThread` to
keep the blocking sensor poller from blocking twisted, and also returning a `deferred` from `LoopingRead`
to ensure twisted will not re-schedule the method until the sensor read is done. The other interesting
bit is using a `deque` as a circular buffer to calculate the rolling average of the sensor readings over
the past n readings. Then, once every 10 seconds, the rolling average is calculated, and shipped off to Redis.

With the sensor polling added now, Python sits at about 10% CPU usage, with SPI writing to the display taking
about 6%. These numbers still seem acceptable.

However, one thing I have noticed with these DHT11s is that it doesn't seem like the temperature changes all
that much. Although, I'm not sure if that's an actual thing, or just lack of resolution on this sensor. The
temperature does change when I breathe on the sensor, so I know it's working. I'll probably spring for a
more expensive DH22, or perhaps use the DH11 for humidity, and get a DS18B20 or something. Either way, there's
a lot of other work to do before I hook this thing up to my thermostat.
