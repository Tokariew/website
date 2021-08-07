---
title: "Audio Rant"
date: 2020-09-17T16:22:03+02:00
url: /audio-rant/
categories:
  - Computers
tags:
  - audio
  - codecs
  - rant
  - computers
draft: true
---

Yesterday I was browsing audio forum, and I stumbled upon 2 threads that irritated me. First thread about how to configure foobar2000 to get better sound, and second was general PC audio playback, but was on heavy side why wave files are better than flacs, and that ripped Audio CD sound worse than playing CD directly.

So let's begin with is there are some reason why flacs can sound differently than waves?

Flacs after decoding will be bit to bit identical with wave files from which they there encoded. So the main difference will be additional load during the decoding phase, so what's the cost of the decoding flacs?
I can test this on three machines, my PC with Ryzen 3600, Raspberry Pi version 3, and microserver with Intel i5 4690k.

So on Ryzen:

    tokariew  …/dys/sol/2019-compilation-2019  time flac -ts 09-lost-in-decline.flac

    real	0m1.310s
    user	0m1.246s
    sys	0m0.037s

On Raspberry:

    pi@raspberrypi:~/Downloads $ time flac -ts 09-lost-in-decline.flac

    real	0m10.187s
    user	0m9.956s
    sys	0m0.220s

And i5:

    [tokariew@nas tmp]$ time flac -ts 09-lost-in-decline.flac

    real	0m1.290s
    user	0m1.256s
    sys	0m0.027s

These are decoding speed using the official flac tool, and the test file was over 17 minutes in length, and with 24 bit depth and 44100 sampling rate. Decoding speed is from 100× to around 800× for this file. So for even some low-end hardware like Raspberry Pi, to decode in real-time we need around 1% of single core. Some tables with decoding speed for portable players is available here[^1].

[^1]: https://www.rockbox.org/wiki/CodecPerformanceComparison

So if less than 1% of additional load on PC should somehow change how audio is played so nearly anything we do on PC should change audio signature. User on thread said that more complex codec sound even worse…
To take this to extreme: if you play audio on PC, you shouldn't do anything else, otherwise sound will be distorted proportional to load on machine. This is absurd. Sound will be perfectly normal up to the moment that computer is at very high load, and then audio will stutter, and you will have problems moving windows on screen…

## Configuring foobar

In thread about configuring foobar for better sound, some advice was about removing some interface from foobar like volume adjustment, progress bar, analyser, and so on… Again to reduce ‘mystical’ and ‘harmful’ load.
Foobar is already pretty minimalistic player, but if we go to such length, maybe we should go and use some console base players, like [musikcube](https://musikcube.com/).
Not on Windows, we lack important bit perfect audio playback… But what is bit perfect playback?

In most systems nowadays, we can have multiple sound sources trying to play at once. To do it, the system mixes sound together to match bit depth and sample rate, and then send it to sound card. Otherwise, we can't get sound notification from programs, when we watch film or listen to sound. So some audio players directed to audiophiles offer bit perfect mode that skip system mixer and send decoded audio stream to sound card. So audiophiles can enjoy their music without being modified by system.

Some caveats when playing bit perfect audio…

-   no ability to change volume on system… So hopefully you can change volume on your playback device, otherwise enjoy full blast.
-   only one audio source can play at once – You are browsing web, and there is some video? You can't probably watch it, even without sound, before stopping audio playback. Depending on audio player, if there is some paused video in browser you can't resume listening to music… You must stop it.
-   You want to use equalizer to somewhat change audio? It will no longer be kosher bit perfect…
-   You have some soundtracks from games? You probably will get some error when trying to play it, because most sound card nowadays support only 44.1 or 48 kHz sample rate and multiples of those rates. So you can't listen without resampling them, so no bit perfect in those case.
-   Your audio collection have audio in 44.1 and 48 kHz sample rate? You will get probably some silence between tracks when switching, or first few seconds of tracks will be silent, then sound card switch between these rates.
-   You have some multichannel audio? Some sound cards will ignore channels except for stereo when in bit perfect mode, you should downmix it into stereo first.
-   Want to play some games when listening to your favourite toons? Most games will not start, or will crash.

So the question is can you listen to audio with these limitations? Will you hear some noticeable difference between bit perfect playback and standard?

I know that on Linux I can't hear the difference between using bit perfect Alsa playback and PulseAudio with resampling. Currently I don't have Windows installed, so I can't test on it.

If I would hear the difference, I would probably use resampling option on foobar, to match system output sample rate. This will guarantee that system don't resample audio from foobar, but still audio can be changed by system if audio volume will be changed.
