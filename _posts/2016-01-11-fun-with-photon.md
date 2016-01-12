---
layout:     post
title:      fun with photon
date:       2016-01-11 20:38:00
summary:    HTTP-triggered sound generator.
---

![](/images/IMG_20160111_222618_edit.jpg){:width="100%"}

I love playing with electronics. Especially with Photon, the latest addition to my collection of Arduino-compatible boards. The best thing about this board is the built-in Wi-Fi module, which gives the Arduino platform wings.

The project I would like to build today is an http-triggered sound generator, which will possibly turn into something bigger in the future. But let's focus on basics first.

The rules of the project are simple:

* the board should listen to HTTP commands and:
  * emit a sound when the command is "on"
  * stop the sound when the command is "off"

There is no need to build something too complicated, so today I'm focusing on two standard functions most Arduino-compatible boards support:

* [tone](https://www.arduino.cc/en/Reference/Tone)
* [noTone](https://www.arduino.cc/en/Reference/NoTone)

Here's the code:

{% highlight c %}
// -----------------------------------------
// Controlling the speaker over the Internet
// -----------------------------------------

int tonePin = D1;

void setup()
{
    Spark.function("sound", soundToggle);
}

void loop() { }

int soundToggle(String command)
{
    int result = -1;

    if (command == "on")
    {
        tone(tonePin, 440);

        result = 1;
    }
    else if (command == "off")
    {
        noTone(tonePin);

        result = 0;
    }

    return result;
}
{% endhighlight %}

You can call the board using your favorite REST tool using the following URL:

```https://api.particle.io/v1/devices/your-device-ID-goes-here/sound?access_token=your-access-token-goes-here```

![](/images/Screenshot from 2016-01-11.png){:width="100%"}

So simple!

The code is also available [here](https://github.com/PiotrJustyna/photon-sandbox/tree/master/5_REST_Sound).
