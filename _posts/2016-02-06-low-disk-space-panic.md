---
layout:     post
title:      low disk space panic
date:       2016-02-06 13:18:00
summary:    Running out of disk space!
---

As a long time Windows user, seeing this:

![low disk space](/images/low-disk-space.png)

slightly freaked me out. I have been using Elementary OS for a while now and thought it would always be just smooth sailing. Nope, not the case. Where did all that space go? Let's find out.

After a couple of minutes of googling, I found this command: **du**. Very good answer which I could apply immediately was here: [unix exchange](http://unix.stackexchange.com/a/125433/155264).

{% highlight bash %}
du -h / | grep '^\s*[0-9\.]\+G'
{% endhighlight %}

And of course, Steam, photos and downloads are to blame. Here's a partial output of this command:

![du scan](/images/du-scan.png)

I'm going to leave Pillars of Eternity installed. Who knows, maybe some day I'll get back to it. But photos can move to an external drive and I don't need the downloads anymore.

Disk cleaned up, now I can write some code!

{% if post.comments %}
<div id="disqus_thread"></div>
<script>
    (function() {  // DON'T EDIT BELOW THIS LINE
        var d = document, s = d.createElement('script');

        s.src = '//piotrjustyna.disqus.com/embed.js';

        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
{% endif %}
