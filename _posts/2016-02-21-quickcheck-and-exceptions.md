---
layout:     post
title:      quickcheck and errors
date:       2016-02-21 18:44:00
summary:    Pinning fussy smart constructors with quickcheck properties.
---

Almost done with my post-refactoring ([roller](https://github.com/PiotrJustyna/roller), smart constructors) changes. The last test I'm adjusting to fit into the new, smart constructor world depends on the smart constructor returning an error if the input data is incorrect. Before smart constructors I relied on roller's parsing code to never cause errors. Worst case scenario it would return **Nothing**, but no errors. This time around, potentially correct input (potentially as passing the parsing stage) can trigger the smart constructor's data checking alarm and, as a result, cause the code to return errors.

Example:

**"123d321"** is technically a parsable string, but my fussy smart constructors will definitely not like it as the limit for both **NumberOfDice** and **NumberOfFacesOfEachDie** is 99.

I already have one property verifying the correctness of parsing valid dice expressions:

{% highlight haskell %}
prop_ParseDieTermGivenCorrectDieTermText :: CorrectDieTermGenerator -> Bool
prop_ParseDieTermGivenCorrectDieTermText (CorrectDieTermGenerator text) =
	maybeDiceExpressionDestructor
		(\numberOfDice numberOfFacesOfEachDie -> numberOfDice >= 0 && numberOfDice <= 99 && numberOfFacesOfEachDie >= 0 && numberOfFacesOfEachDie <= 99)
		ignoredDieTerm
		ignoredDieTerm
		ignoredConstantTerm
		ignoredConstantTerm
		ignoredConstantTerm
		False
		(text =~ dieTerm)
	where
		ignoredDieTerm = \_ _ -> False
		ignoredConstantTerm = \_ -> False
{% endhighlight %}

But there are so many things that can should be covered to form a complete suite of negative cases:

* try parsing unparsable text - **Nothing** is returned
* try parsing text violating the **NumberOfDice** restriction (99) - results in an error
* try parsing text violating the **NumberOfFacesOfEachDie** restriction (99) - results in an error
* try parsing text violating both **NumberOfDice** and **NumberOfFacesOfEachDie** restrictions (both 99) - results in an error

As you noticed, I added one more argument to my destructor to cater for scenarios where returning **Nothing** does not mean that the destructor should automatically return **False**. New destructor code looks like this now:

{% highlight haskell %}
maybeDiceExpressionDestructor :: (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (NumberOfDice -> NumberOfFacesOfEachDie -> Bool) -> (Word8 -> Bool) -> (Word8 -> Bool) -> (Word8 -> Bool) -> Bool-> Maybe DiceExpression -> Bool
maybeDiceExpressionDestructor dieTerm addedDieTerm subtractedDieTerm constantTerm addedConstantTerm subtractedConstantTerm nothingResult x = case x of
  Just diceExpression -> diceExpressionDestructor dieTerm addedDieTerm subtractedDieTerm constantTerm addedConstantTerm subtractedConstantTerm diceExpression
  Nothing -> nothingResult
{% endhighlight %}

---

At this stage I realised that I had to do some reading as verifying the failures (errors) in smart constructors was getting really tricky - I couldn't determine the return type as returning errors simply escape the control structures.

I found [this article](https://wiki.haskell.org/Error_vs._Exception) and suddenly errors and exceptions started making more sense. It doesn't look like using "errors" in smart constructors makes much sense as data validation failures are expected and do happen. As a result, exceptions looks like a much more suitable option. However - is throwing exceptions really the best course of action?

[Gabriel Gonzalez](http://stackoverflow.com/a/10168184/224612) was kind enough to suggest using Liquid Haskell to validate input in my scenario and I think I will educate myself a bit before attempting to solve my problem with errors in smart constructors.

However, for now I think the smart constructors are a good first step when it comes to input data validation and new version of the library should be released to reflect those changes.

You can find the latest changes I have been writing about in my [roller](https://github.com/PiotrJustyna/roller/commit/cffb4b5a7381bbba7272fd0a3696f696cc9a892d) repo now and please feel free to test the new version of the library by downloading it from [Hackage](https://hackage.haskell.org/package/roller-0.1.7).

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
