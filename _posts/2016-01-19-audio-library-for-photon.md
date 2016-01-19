---
layout:     post
title:      audio library for photon
date:       2016-01-15 20:46:00
summary:    Audio! Photon! Adventure!
---

Great thing about Photon is its fantastic [cloud IDE](http://build.particle.io/). However, if you're used to playing with exotic Arduino libraries and if they are not ported, you're on your own. This is the scenario in my case - AVR libraries (which I need to make my Photon play chiptunes) are not included in the Photon libraries. Not to worry, though, let's see what we can do.

First of all, the code I will be porting is available here: [8-bit, 8000 Hz audio playback](http://playground.arduino.cc/Code/PCMAudio).

The AVR libraries should be available in the Adruino distribution package. Let's check... Downloading... Extracting... Searching... Success!

![AVR](/images/Screenshot from 2016-01-15.png)

Now I have to play a bit with references to the AVR header files. Back in a bit!

Wait... Photon is **ARM**, not **AVR**.

![Facepalm](/images/facepalm-ernie.jpg)

Wait... I have a [Teensy board](http://www.pjrc.com/store/teensy32.html), which is **ARM**. Is there hope for me?

[Let's do some reading first](https://github.com/PaulStoffregen/Audio).

It looks like [Paul Stoffregen](https://github.com/PaulStoffregen) (creator of Teensy) did some fantastic work to build the Teensy audio library. Looking at the amount of code, though, porting it to Photon is not going to be an easy task.

Let's get started...

...

...

...

...after a couple of hours spent on porting the library and all its dependencies I decided to do a bit of searching. But not on google - on the photon community forum. Guess what I found there. Yes, there already is a (super compact) audio library! A gentleman called [ensonic](https://github.com/ensonic) was kind enough to put the code together to form a really lightweight utility for playing audio on Photon. Well done, ensonic, and thank you!

[https://github.com/ensonic/photon-waveout](https://github.com/ensonic/photon-waveout)

The code is tested and I confirm it's working on my Photon. The sound is a bit jagged and the quality is not there yet, but this is all due to the fact (I hope!) the circuit is just a raw output to randomly found earphones.

This is probably not my last adventure with Photon audio, but that's a story for another day.

Happy hacking!
