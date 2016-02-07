---
layout:     post
title:      smart destructors
date:       2016-02-07 17:01:00
summary:    Smart constructors & pattern matching.
---

During the last couple of days I didn't have a chance to do much programming in my spare time and started really missing Haskell. Just a quick reminder: the code I'm working on is the [roller](https://github.com/PiotrJustyna/roller) library which I'm using to get familiar with the language. If you see something that can be done better, please feel free to comment! Let's see where I finished the last time...

Oh, ok. I refactored one suite of properties and now it's time for the remaining tests to follow. The suite I fixed last time was "TypesTests", now it's time to address all compilation problems in the "ParseTests" caused by introducing smart constructors. Let's first see what is breaking:

{% highlight bash %}
$ ghc -package-db=.cabal-sandbox/x86_64-linux-ghc-7.8.3-packages.conf.d Tests/ParseTests.hs -o bin/parsetests
[3 of 3] Compiling Main             ( Tests/ParseTests.hs, Tests/ParseTests.o ) [Roller.Parse changed]

Tests/ParseTests.hs:69:11: Not in scope: data constructor ‘DieTerm’

Tests/ParseTests.hs:78:11: Not in scope: data constructor ‘DieTerm’
{% endhighlight %}

OK, two erros. This is caused by the constructors not being visible anymore (smart constructors are used instead). Let's see the test code:

{% highlight haskell %}
prop_ParseDieTermGivenCorrectDieTermText :: CorrectDieTermGenerator -> Bool
prop_ParseDieTermGivenCorrectDieTermText (CorrectDieTermGenerator text) =
  case (text =~ dieTerm) of
    Just (DieTerm x y) ->
			x >= 0 && x <= 99
			&&
			y >= 0 && y <= 99
    Nothing -> False

prop_ParseDieTermGivenIncorrectNumberOfDiceDieTermText :: IncorrectNumberOfDiceDieTermGenerator -> Bool
prop_ParseDieTermGivenIncorrectNumberOfDiceDieTermText (IncorrectNumberOfDiceDieTermGenerator text) =
  case (text =~ dieTerm) of
    Just (DieTerm x y) -> False
    Nothing -> True
{% endhighlight %}

Oh no!

![smart constructors](/images/yfdmo.jpg)

Googling...

[As it turns out](http://stackoverflow.com/a/10168184/224612), there is a way to have smart constructors and pattern match on the regular constructors and it is to use smart destructors (or [Church encoding](https://en.wikipedia.org/wiki/Church_encoding), if more familiar with lambda calculus than me).

OK, let's try to use this in action.

This is my type with all its constructors:

{% highlight haskell %}
data DiceExpression =
    DieTerm NumberOfDice NumberOfFacesOfEachDie
    | AddedDieTerm NumberOfDice NumberOfFacesOfEachDie
    | SubtractedDieTerm NumberOfDice NumberOfFacesOfEachDie
    | ConstantTerm Word8
    | AddedConstantTerm Word8
    | SubtractedConstantTerm Word8
{% endhighlight %}

I am not too sure a type definition should look like this, as the linked example looked slightly different, but it was already like this when I took ownership of the library and don't want to make too many breaking changes at once. In my case, smart destructor would look something like this:

{% highlight haskell %}
diceExpressionDestructor :: (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (Word8 -> Bool) -> (Word8 -> Bool) -> (Word8 -> Bool) -> DiceExpression -> Bool
diceExpressionDestructor dieTerm addedDieTerm subtractedDieTerm constantTerm addedConstantTerm subtractedConstantTerm x = case x of
  DieTerm numberOfDice numberOfFacesOfEachDie -> dieTerm numberOfDice numberOfFacesOfEachDie
  AddedDieTerm numberOfDice numberOfFacesOfEachDie -> addedDieTerm numberOfDice numberOfFacesOfEachDie
  SubtractedDieTerm numberOfDice numberOfFacesOfEachDie -> subtractedDieTerm numberOfDice numberOfFacesOfEachDie
  ConstantTerm n -> constantTerm n
  AddedConstantTerm n -> addedConstantTerm n
  SubtractedConstantTerm n -> subtractedConstantTerm n
{% endhighlight %}

Wow. That's certainly a long type declaration. Now I just have to expose the function... done, and start testing.

{% highlight haskell %}
print . show $ diceExpressionDestructor
  (\x y -> False)
  (\x y -> False)
  (\x y -> False)
  (\x -> False)
  (\x -> True)
  (\x -> False)
  (constructAddedConstantTerm 4)
{% endhighlight %}

And this returns **True** when run. OK, it looks very promising - the destructor makes the decision which constructor to pattern match the {% highlight haskell %}constructAddedConstantTerm 4{% endhighlight %} against. Now how do I effectively use it with my QuickCheck properties? Let's see...

I am going to start with the first failing property:

{% highlight haskell %}
prop_ParseDieTermGivenCorrectDieTermText :: CorrectDieTermGenerator -> Bool
prop_ParseDieTermGivenCorrectDieTermText (CorrectDieTermGenerator text) =
  case (text =~ dieTerm) of
    Just (DieTerm x y) ->
			x >= 0 && x <= 99
			&&
			y >= 0 && y <= 99
    Nothing -> False
{% endhighlight %}

And here's another challenge: my destructor can decompose an instance of DiceExpression, but not an instance of Maybe DiceExpression. Not a big problem, though. Let's add another destructor that can do it:

{% highlight haskell %}
maybeDiceExpressionDestructor :: (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (Word8 -> Bool) -> (Word8 -> Bool) -> (Word8 -> Bool) -> Maybe DiceExpression -> Bool
maybeDiceExpressionDestructor dieTerm addedDieTerm subtractedDieTerm constantTerm addedConstantTerm subtractedConstantTerm x = case x of
  Just diceExpression -> diceExpressionDestructor dieTerm addedDieTerm subtractedDieTerm constantTerm addedConstantTerm subtractedConstantTerm diceExpression
  Nothing -> False
{% endhighlight %}

Done! Now let's use it in my property:

{% highlight haskell %}
prop_ParseDieTermGivenCorrectDieTermText :: CorrectDieTermGenerator -> Bool
prop_ParseDieTermGivenCorrectDieTermText (CorrectDieTermGenerator text) =
	maybeDiceExpressionDestructor
		(\numberOfDice numberOfFacesOfEachDie ->
			numberOfDice >= 0 && numberOfDice <= 99
			&&
			numberOfFacesOfEachDie >= 0 && numberOfFacesOfEachDie <= 99)
		ignoredDieTerm
		ignoredDieTerm
		ignoredConstantTerm
		ignoredConstantTerm
		ignoredConstantTerm
		(text =~ dieTerm)
	where
		ignoredDieTerm = \_ _ -> False
		ignoredConstantTerm = \_ -> False
{% endhighlight %}

And it's working!

{% highlight haskell %}
"Verify parse Die Term given correct die term text."
+++ OK, passed 100 tests.
{% endhighlight %}

And there we go! I have smart constructors and pattern matching, which is super useful when it comes to testing the code. Now I just have to apply the same mechanism to all broken properties and I think I will release new version of roller.

As I build the library, I am adding more and more code to the Types module I am not sure this is the right thing to do. At the moment I don't see any better way to achieve my goals but as this is a learning process, there probably will be a lot of refactoring as I start seeing my mistakes. For now the code is working predictably and the test coverage starts shaping up, so I couldn't be happier.

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
