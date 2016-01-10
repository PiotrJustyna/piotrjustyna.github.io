---
layout:     post
title:      continuing the cleanup
date:       2016-01-10 20:13:00
summary:    Post-refactoring test updates.
---

![](/images/tea.jpeg){:height="30%" width="30%"}

This piece of work I left specifically for Sunday. Nice, relaxed evening, tea and gentle ambient music are perfect for updating broken tests.

As you remember, all started when I decided to use smart constructors in my [roller](https://github.com/PiotrJustyna/roller) repository as a way of ensuring the input data for constructors meets my criteria. This broke some tests, I fixed one of them last night and now, since I know what broke and how to fix it, have to fix remaining tests.

At the time of writing this, I have two test files:

* TypesTests.hs
* ParseTests.hs

I decided to start with TypesTests as the logic there is more straightforward.

Remaining properties to fix there are:

* ```prop_ShowAddedDieTerm```
* ```prop_ShowSubtractedDieTerm```
* ```prop_ShowConstantTerm```
* ```prop_ShowAddedConstantTerm```
* ```prop_ShowSubtractedConstantTerm```

First two properties are easy to fix as the only thing that really needs changing is the input parameters (generators should be used). The other two are not much more complicated - they just require new generator (this time just one).

Initial code:

{% highlight haskell %}
prop_ShowConstantTerm :: Word8 -> Bool
prop_ShowConstantTerm x = show (constructConstantTerm x) == show x
{% endhighlight %}

Error:

{% highlight bash %}
"Verify show ConstantTerm."
*** Failed! (after 21 tests and 4 shrinks):                               
Exception:
  Constat incorrect.
  Details:
  Given constant: 100 (limit: 99).
100
{% endhighlight %}

The solution is to write a showable constant generator:

{% highlight haskell %}
newtype CorrectConstatntGenerator = CorrectConstatntGenerator Word8 deriving Show
instance Arbitrary CorrectConstatntGenerator where arbitrary = CorrectConstatntGenerator <$> generateCorrectConstant

generateCorrectConstant :: Gen Word8
generateCorrectConstant = elements [0 .. 99]
{% endhighlight %}

And then to just use it in all three "ConstantTerm" properties (here, showing only the first one):

{% highlight haskell %}
prop_ShowConstantTerm :: CorrectConstatntGenerator -> Bool
prop_ShowConstantTerm (CorrectConstatntGenerator x) = show (constructConstantTerm x) == show x
{% endhighlight %}

Does it work? Let's find out:

{% highlight bash %}
"Verify show ConstantTerm."
+++ OK, passed 100 tests.
{% endhighlight %}

Success!

Now, since I have all TypesTests properties updated and working with smart constructors, I should check and update ParseTests properties, but it's a task for another day.

For now, I'm just happy to see this:

{% highlight bash %}
"Verify show DieTerm."
+++ OK, passed 100 tests.
"Verify show AddedDieTerm."
+++ OK, passed 100 tests.
"Verify show SubtractedDieTerm."
+++ OK, passed 100 tests.
"Verify show ConstantTerm."
+++ OK, passed 100 tests.
"Verify show AddedConstantTerm."
+++ OK, passed 100 tests.
"Verify show SubtractedConstantTerm."
+++ OK, passed 100 tests.
{% endhighlight %}
