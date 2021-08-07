---
title: "AudioFool"
date: 2020-09-16T18:17:26+02:00
url: /AudioFool/
categories:
  - Audio
tags:
  - audio
  - Windows
  - codecs
  - more tags
  - simple
draft: true
---
Today I was browsing audio forum, and i stumbled on thread about why JRiver is better player than foobar, from there I got into thread how to configure foobar player for better sound, and then about why flacs are inferior to wave files.
<!--more-->

I was little terrified with amount of magic thinking and disinformation in this threads…
So let's clear some misconceptions

## Flacs sound worse than wave files

Someone consider flacs to be worse sounding because PC need to decode flacs into pcm, and becouse of it sound is "thinner".

So first let's look how long my PC need to decode one of my longest audio files in collection (over 17 minutes long) using sox.

```bash
> time sox 09-lost-in-decline.flac out.wav

real	0m1.621s
user	0m1.458s
sys	0m0.126s
```

So track over 17 minutes is decoded in about 1.6 seconds. The flac was encoded with compression level 8, highest possible. Let's take simplistic approach. Program start playback, during first 2 seconds of playing, flac is fully decoded, and any "thinner" sound should dissapear, because we have now wave file.

Still author of thread disgard that saying that he have some ancient laptop with Intel processor. So let's test decoding speed on raspberry pi 3.
```bash
pi@raspberrypi:~/Downloads $ time sox 09-lost-in-decline.flac out.wav

real	0m16.546s
user	0m13.964s
sys	0m1.903s

```

So the same file was decoded in 17 seconds.

```bash
> time sox 09-lost-in-decline.mp3 out2.wav

real	0m2.368s
user	0m2.264s
sys	0m0.074s
```
