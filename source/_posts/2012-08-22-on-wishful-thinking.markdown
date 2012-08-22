---
layout: post
title: "On Wishful Thinking"
date: 2012-08-22 11:21
comments: true
categories: 
---

I love [Raymond Chen's blog](http://blogs.msdn.com/b/oldnewthing/), especially when he goes thermonuclear on his customers (or readers) for asking stupid questions or trying to go off topic or, you know, just plain trolling.

Something I find funny about the highly technical posts on his blog is that Mr Chen usually supplements his explanations with witty remarks like "the customer managed to screw it despite this being clearly documented here and here and there." Well of course he is being polite, since it should surprise no one the fact that most people just do not care about documentation or doing things the right way in general.

You see, a lot of companies employing programmers (I dare say most of them) are usually not that interested in the long lasting quality of internal software products. If, as a developer, you know your application will have to be ready in less than 2 weeks, will be used only by a handful of people with pretty much identical skill set and low self-esteem, you won't lose that much time in making sure your solution is absolutely the best anyone can come up with. You won't care if it's efficient, with great performance and minimum memory consumption, or if it has a usable user interface.

When a programmer finds an error the usual way to solve it goes like this:

<a href="http://www.flickr.com/photos/wishfulcoding/6315168390/"><img src="http://farm7.static.flickr.com/6032/6315168390_be3136771f.jpg" width="100%" alt="Wishful Thinking"></a>

So, this marvellous reasoning means that most of the time we won't slow down to understand what's going on, what went wrong and how to solve it nicely. It means that by sheer luck sometimes we'll come up with a working solution, and most of the time we'll create a time bomb. The code will work ok on our machines under very specific circumstances, and then it will fail spectacularly on a live environment or down the road when some of the specs/settings change in an unexpected way. And the whole cycle will start again. Most of the time it won't even get to this point, because the application will be decommissioned, or because we will have moved on to a different project/department/company.

I think of a particular aspect of this reasoning, the "Let's try the first thing that comes to my mind and see if it works", as a special case of wishful thinking I like to call _Wishful Coding_. It basically involves trying to adapt programming tools and languages to our mental model, because "That's how I imagine this class or function should behave so let's just assume they will and hope for the best."

Wishful coding is the reason why [The Daily WTF](http://thedailywtf.com/) continues to thrive and why there's a whole subculture of mocking catastrophically funny pieces of code. It's especially bad on languages with unsafe constructs (like C or C++) which don't offer enough safety nets by default, such as garbage collection or boundary checks. If you can cause any harm to yourself or society, then for sure you will.

So, is there anything we can do to improve this bleak landscape? Of course there is. A real life version of the [Joo Janta 200 Super-Chromatic Peril Sensitive Sunglasses](http://en.wikipedia.org/wiki/Technology_in_The_Hitchhiker%27s_Guide_to_the_Galaxy#Joo_Janta_200_Super-Chromatic_Peril_Sensitive_Sunglasses) [may or may not be in the works](http://en.wikipedia.org/wiki/Project_Glass), so I would keep an eye on those.

We can also try scaring people into doing things right, despite pressure from management. Somebody said _"Always write code as if the person who will be maintaining it is a psychotic serial killer who knows where you live."_ So we could try, say, hiring trainee programmers with actual criminal records and give them the rest of the staff's home addresses.

A different approach which seems to be quite popular these days is making difficult for people to shoot themselves (and others) in the feet. Joel Spolsky says that [_Something is usable if it behaves exactly as expected._](http://www.joelonsoftware.com/design/1stDraft/03.html). That also seems to apply to our development tools. Emerging frameworks and languages appear to focus in allowing programmers to [express intent](http://www.hanselman.com/blog/ProgrammerIntentOrWhatYoureNotGettingAboutRubyAndWhyItsTheTits.aspx) instead of complying with the strict rules of machine language. They favour convention over configuration, readability over conciseness, security over performance. Joe Coder shouldn't be spending any time in deciding what the best technique is for concatenating immutable strings in a loop, just let the platform take care of those details.

Since we already decided to follow this path then why not going all the way down? Why not deprecate technologies that could potentially allow anyone to cause any kind of harm? Sorry, C and C++ and Objective C: You have to go. CGI? OpenGL? Don't even get me started. Root access? HA! We should keep dumbing down our tools, so let's make it hard for new programmers to overflow a stack, to divide by zero, to cause a deadlock. Let's ban kernel access, unsafe pointers and unmanaged memory. Everyone embrace sandboxed languages: Let's write a H.264 decoder in JavaScript!!! ([Oh, wait...](http://arstechnica.com/open-source/news/2011/10/native-javascript-h264-decoder-offers-compelling-demo-of-js-performance.ars))

Seriously now, it's obvious that current trends on corporate IT will surely continue for decades and most legacy systems will actually outlive us. But at least there are now more chances than ever for the next generation of wishful coders to empower their businesses without creating maintenance nightmares in the process. Every year we are closer and closer to intuitively create robust, scalable systems without falling in an endless cycle of corruption and despair.

Of course, it's also possible that future programmers will despise our feeble attempts at improving the Status Quo, decide that our tools are too narrow and restrictive, and end up exploiting all kinds of quirks to work around the platform limitations. And we'll be back at the starting line.

Nobody knows. And since we suck at predicting the future, maybe [it doesn't even matter](http://neo.jpl.nasa.gov/risk/). We'll see. 

