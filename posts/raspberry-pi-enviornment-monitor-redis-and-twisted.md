Title: Raspberry Pi Environment Monitor - Twisted Redis
Category: RaspberryPi

[In the first part]({filename}raspberry-pi-enviornment-monitor-intro.md), I settled on using the Twisted
framwork in Python, along with using Redis as a data store. There's a Redis library for Python, but I
was unsure how to tie that into Twisted. But, it turns out that, once again, someone has done the hard
work with the [txRedis](https://github.com/deldotdr/txRedis) library.

My first goal is writing a wrapper around txRedis to expose the functionality I need. In the process
of learning txRedis, I learned about the inlineCallbacks functionality in twisted that either wasn't
there the last time I did something with Twisted (at least 3 years ago), or I ignored then. It seemed
odd at first, until I realized it was essentially the async/await pattern in C#.

For my wrapper, I want to do the following things:

*   Store a dictionary, returning a unique key
*   Add a key to a sorted set, with the score being the current time.
    
    This will allow me to query a list of objects within a store during a time frame (say, the last 10
    minutes). Which leads to the next function.

*   Query a store for items during the last n minutes.
*   Publish a new item for consumers to act on
*   Listen for new published items

This, with the exception of the subscribing, is implemented thusly:

<script src="https://gist.github.com/msieker/932b90e1ec67855a76e9.js"></script>

The `settings` object passed into the `__init__` is a dictionary created by merging together a pair
of yaml files. One for non-private data to act as a template, the other for private data (API keys,
locations, and so on)

<script src="https://gist.github.com/msieker/93f7876d349d9ab4626c.js"></script>

With this, the basic outline of how I would like my redis interface to look like is complete. Next
up is the first consumer of this class: The weather logger.