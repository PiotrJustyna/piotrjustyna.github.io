---
layout:     post
title:      haskell & smart constructors
date:       2016-01-08 23:10:00
summary:    Safety, constraints and comforting errors.
---

Work is stressful and mostly chaotic. Logic and structure rarely applies in this realm. This is why playing with Haskell after hours feels like a blessing. Haskell code is comfortingly safe, clean and compact. It's relatively difficult to break things thanks to the rock solid type system.

I've been learning and playing with Haskell on and off for over a year now and I still feel like an absolute beginner. Problems I face in my daily work, like validation of parameters passed into constructors, are not as straightforward to solve in Haskell (at least for a noob like me). To make sure your constructor input data is correct, [smart constructors](https://wiki.haskell.org/Smart_constructors) are often a suggested option.

[Roller](https://github.com/PiotrJustyna/roller), a repository I use to learn Haskell lets me face those (seemingly trivial) issues and scratch my head wondering how to approach them. After a couple of days of intense googling and refactoring I managed to redesign a part of the code to support smart constructors. I even broke my quickcheck properties (for the right reason!). It feels so safe now.

Yay!

{% highlight bash %}
piotr@mancave:~/Documents/Projects/roller$ ./bin/typestests
"Verify show DieTerm."
*** Failed! (after 21 tests and 6 shrinks):                               
Exception:
  Number of dice or number of faces of each die incorrect.
  Details:
  Given number of dice: 0 (limit: 99).
  Given number of faces of each die: 100 (limit: 99).
0
100
{% endhighlight %}

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
