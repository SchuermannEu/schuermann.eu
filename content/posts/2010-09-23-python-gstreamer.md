---
layout: post
title: Load any soundfile in Python as Scipy array using GStreamer
author: Dominik Schürmann
date: 2010-09-23
slug: python-gstreamer
aliases:
    - "/2010/09/23/python-gstreamer.html"
---

I had problems using the build in function of Scipy to load soundfiles as an scipy.array object. Sometimes it gives an error like “struct.error: unpack requires a string argument of length 4″ when opening some wave files. Yes it only handles wave files!

After trying [audiolab](http://www.ar.media.kyoto-u.ac.jp/members/david/softwares/audiolab/), which was not working, and the build-in functions for opening wave files I stumbled upon [“Use (Python) Gstreamer to decode audio (to PCM data)”](http://stackoverflow.com/questions/3507746/use-python-gstreamer-to-decode-audio-to-pcm-data). Because I was already using GStreamer in this project to handle microphone recordings I used the [linked library](http://github.com/sampsyo/pylastfp/blob/master/lastfp/gstdec.py) from the post and wrote a little function to convert the raw PCM data to readable scipy arrays. It was a little bit tricky to get the struct.unpack right but now it works like a charm.

So now we have a working implementation that can convert any audiofile, that gstreamer can open, to a scipy.array. Thanks goes to [Adrian Sampson](http://github.com/sampsyo) for providing the decoding function.

See my changes on [http://gist.github.com/592776](http://gist.github.com/592776)
