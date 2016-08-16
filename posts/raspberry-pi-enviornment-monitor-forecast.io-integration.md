Title: Raspberry Pi Environment Monitor - Forecast.io Integration 
Category: RaspberryPi

[With Redis support in place]({filename}raspberry-pi-enviornment-monitor-redis-and-twisted.md), it's
time to create the first thing to write into the store: The weather forecasts. 

As stated earlier, I decided on using [forecast.io](http://forecast.io) due to their easy (and free
for the most part) API. Most of the code for the forecast downloader involves talking to Redis or 
dealing with Twisted.

<script src="https://gist.github.com/msieker/0d6a1e8fc36858a19a12.js"></script>

Originally I was going to exclude the daily forecast bits from the API call, but then I noticed by
reading the docs that some interesting bits of information are in there, like sunrise and sunset times,
along with daily highs and lows. As I'm not too interested in displaying an extended forecast on this
thing, I'm just grabbing basic data I'm interested in from the first daily data element returned, which
is the current day.

Once I have the data I'm interested in, I push it into Redis, add it to the time series sorted set,
and push out to subscribers that there's been an update.

Now that I have data, I need to actually do something with this data. To the display!