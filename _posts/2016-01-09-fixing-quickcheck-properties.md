---
layout:     post
title:      fixing quickcheck properties
date:       2016-01-09 20:42:00
summary:    Tests failed, time to tweak them.
---

![](/images/CYPIqq9WcAEZp85.jpg)

Code changed, tests failing, fantastic. Now let's fix them*.

First of all, let's see what exactly is breaking:

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

Looking at the property's code:

{% highlight haskell %}
prop_ShowDieTerm :: Word8 -> Word8 -> Bool
prop_ShowDieTerm x y =
  show (constructDieTerm x y) == show x ++ show dieSymbol ++ show y
{% endhighlight %}

Nothing too complicated. It is the smart constructor that does not like quickcheck's input. Let's see why. Here's the code:

{% highlight haskell %}
constructDieTerm :: NumberOfDice -> NumberOfFacesOfEachDie -> DiceExpression
constructDieTerm x y
  | validateDieTermParameters x y = DieTerm x y
  | otherwise = error $ dieTermLimitsErrorMessage x y

validateDieTermParameters :: NumberOfDice -> NumberOfFacesOfEachDie -> Bool
validateDieTermParameters x y = x <= diceLimit && y <= facesOfEachDieLimit
{% endhighlight %}

Oh, ok, I see. The error message looks like the 0 number of dice might be the problem but in fact it is the number of faces of each die - it's 100 and the limit is 99. The answer to that is to adjust quickcheck's input data. For this property (*verify that showable DieTerms are shown in a certain way*), I will narrow the input down to values producing showable DieTerms:

* number of dice: 0-99
* number of faces of each die: 0-99

We can do it with [generators](https://hackage.haskell.org/package/QuickCheck-2.7.2/docs/Test-QuickCheck-Gen.html). Generators help provide quickcheck with desired test data input:

{% highlight haskell %}
newtype CorrectNumberOfDiceGenerator = CorrectNumberOfDiceGenerator NumberOfDice deriving Show
instance Arbitrary CorrectNumberOfDiceGenerator where arbitrary = CorrectNumberOfDiceGenerator <$> generateCorrectNumberOfDice

newtype CorrectNumberOfFacesOfEachDieGenerator = CorrectNumberOfFacesOfEachDieGenerator NumberOfFacesOfEachDie deriving Show
instance Arbitrary CorrectNumberOfFacesOfEachDieGenerator where arbitrary = CorrectNumberOfFacesOfEachDieGenerator <$> generateCorrectNumberOfFacesOfEachDie

generateCorrectNumberOfDice :: Gen Word8
generateCorrectNumberOfDice = elements [0 .. 99]

generateCorrectNumberOfFacesOfEachDie :: Gen Word8
generateCorrectNumberOfFacesOfEachDie = elements [0 .. 99]
{% endhighlight %}

Just as a reminder, I'm using following aliases to make the code more explicit:

{% highlight haskell %}
type NumberOfDice = Word8
type NumberOfFacesOfEachDie = Word8
{% endhighlight %}

With generators prepared like this, I can modify the ```prop_ShowDieTerm``` property to operate on them and not on type ```Word8```:

{% highlight haskell %}
prop_ShowDieTerm :: CorrectNumberOfDiceGenerator -> CorrectNumberOfFacesOfEachDieGenerator -> Bool
prop_ShowDieTerm (CorrectNumberOfDiceGenerator x) (CorrectNumberOfFacesOfEachDieGenerator y) =
  show (constructDieTerm x y) == show x ++ show dieSymbol ++ show y
{% endhighlight %}

Does it work? Let's find out:

{% highlight bash %}
"Verify show DieTerm."
+++ OK, passed 100 tests.
{% endhighlight %}

Success!

Now I should fix remaining properties but that's a task for another day.

Other things to consider:

* since I'm using smart constructors now, it would be a good idea to test them
* since smart constructors take care of data checking I don't have to verify the "Show" implementation for incorrect inputs (as every incorrect input will not even make it to the ```show``` function - an error will be generated first)

\**The code I'm writing here may not be immediately available in the [roller](https://github.com/PiotrJustyna/roller) repository (even in the **develop** branch).*

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
