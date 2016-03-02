---
layout:     post
title:      catching exceptions in haskell
date:       2016-03-01 23:15:00
summary:    Haskell and Exceptions - Hello World!
---

I was going to go to bed earlier, but there's that one thing bugging me and it's Haskell exceptions. During one of my previous adventures with Haskell errors I struggled with handling them graciously in quickckeck properties. Then I read about [exceptions vs. errors](https://wiki.haskell.org/Error_vs._Exception), discovered that errors are probably not the best choice in expected, probable scenarios and suddenly wanted to start experimenting with exceptions.

No time to waste, let's do some reading and write a "Hello World!" program for exceptions.

Reading material:

* [HaskellWiki - Exception](https://wiki.haskell.org/Exception)
* [Hackage - Control.Exception](https://hackage.haskell.org/package/base-4.8.2.0/docs/Control-Exception.html)

After some reading I managed to formulate my first exception throwing and catching piece of code:

{% highlight haskell %}
{-#LANGUAGE DeriveDataTypeable#-}

import Control.Exception
import Data.Typeable (Typeable)

data HelloWorldException = HelloWorldException deriving (Show, Typeable)

instance Exception HelloWorldException

troublemaker :: Int -> IO ()
troublemaker x = if x == 0 then throw HelloWorldException else putStrLn $ show x

main :: IO ()
main = do
	catch (troublemaker 0) (\e -> putStrLn $ "Caught " ++ show (e :: HelloWorldException))
	catch (troublemaker 1) (\e -> putStrLn $ "Caught " ++ show (e :: HelloWorldException))
{% endhighlight %}

Output:

{% highlight bash %}
Caught HelloWorld
1{% endhighlight %}

Which does work (well, does throw and does catch...) but I'm yet to discover if this is the way to deal with exceptions. For now it will suffice.

Now to use exceptions in practice! But that's a story for another day.

Happy hacking!

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
