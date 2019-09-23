---
layout: post
author: Marcelo Costa
---
The Nashorn Javascript engine

The Nashorn Javascript engine that will be integrated in the JDK 8 may be, in a near future, the technology that will serve as a foundation for a wide range of enterprise solutions, the increasing adoption of the Javascript language for unorthodox practices, such as, server-side programming, is now a strong topic of discussion among developers, I had the opportunity to attend to Brazil JS in Fortaleza (CE, Brazil) back in 2011 and the whole thing seemed like a Fashion Show to me. Until that event, I didn’t have the chance to experience any of that during my life as an IT professional so I felt like I was just looking at a bunch of crazy trends:

![server-sidejshot](https://themarcelor.github.com/blog/assets/img/server-sidejshot.gif)

My favorite presentation was the one entitled, in portuguese, “Javascript por debaixo dos panos” (which would be something like: Javascript “under the hood” or “behind the curtains”. You got the idea), Douglas Campos was the only one that mentioned the usage of well-consolidated Java libraries running within JVM Javascript engines. I didn’t know about Nashorn back then but the idea of reusing the wheel sounds very attractive. I noticed the giggles around me while he was talking about it, because the _hypsters_ created a bias against Java, or …*Jabol* (since some of them are now referring to it as the new COBOL). Yeah… we have a complete redesign of GC, Project Coin, NIO, Lambdas and there are people that still feel that Java is not evolving. Lexically, perhaps, not that much, but the platform itself have a lot to offer and I believe the multi-language paradigm is yet to show its true power.

So, why Javascript?

It’s … “easy” to use (if you are not using its true syntaxic power): If you ever did some web programming you know the basics to start writing some code and it plays well with a wide-range of “IDEs”: any Notepad++, Emacs, TextMate or Eclipse is enough to get you started.
It’s easily extensible with Object-Oriented-Programming features and prototyping: There are many frameworks that you might have used, e.g., JQuery, Dojo, it’s interesting to see the power of the language when you have a chance to dive in some of their functions.
This last bullet point might seem a little bit subjective but It is naturally bond to the Web: Javascript was created to work with web pages, it was integrated with all browsers, had its engine redesigned by Google (yep, the famous V8 JS Engine), it allowed Microsoft to create something that didn’t crash with a blue screen of death (AJAX) and, in more “recent” developments, has been used in a famous implementation of server-side Javascript, Node.JS.
In summary, studying Nashorn is very fun: exploring the technology, downloading/building the openJDK, running the samples, writing my demo application to test JVM monitoring and, specially, annoying Jim, Marcus and Attila throughout the process.

Yeah, that’s it for today. Stay tuned for more posts about Nashorn.
