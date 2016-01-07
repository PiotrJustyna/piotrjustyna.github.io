---
layout:     post
title:      Hello World
date:       2016-01-07 00:20:00
summary:    Testing my new blog.
---

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

{% highlight haskell %}
appendIfOdd x collection
    | odd x = x : collection
    | otherwise = collection

takeOddl = foldl (\accelerator x -> appendIfOdd x accelerator) []
takeOddr = foldr (\x accelerator -> appendIfOdd x accelerator) []

divideSubsequentl1 = foldl1 (\accelerator x -> x / accelerator)
divideSubsequentr1 = foldr1 (\x accelerator -> x / accelerator)

main = do    
    print (show (takeOddl [1..5])) -- prints "[5,3,1]"
    print (show (takeOddr [1..5])) -- prints "[1,3,5]"

    print (show (divideSubsequentl1 [2, 4])) -- prints "2.0"
    print (show (divideSubsequentr1 [2, 4])) -- prints "0.5"
{% endhighlight %}
