---
layout: post
author: Marcelo Costa
---
From,To = Survivor
Inside the Young area we have 3 smaller areas which are: ‘Eden’, ‘From’ and ‘To’, to illustrate:

young_spaces

Once an object gets created it shoud immediately go to Eden (in case it is too big for the Eden space, it will be automatically promoted to the Old generation, this event is called ‘premature promotion’, that is NOT cool), after a minor collection happens we move only the surviving objects to the first survival area, i.e., the ‘From’ space, after another minor collection, the same ones that survive will be moved to the second area, i.e., the ‘To’ Space, there is no GC in this space, the object is protected against the garbage collection, it will sit in this state until a ‘switch’ happens, in a given moment (whenever the JVM wants) the ‘To’ space will become the ‘From’ space and they will switch places again later over and over.

This might be a little confusing so I will try to make it more complicated through a drawing:

young_explicando

So, the object is created on the ‘Eden’ space, it is moved to the ‘From’ space, and then moved to the ‘To’ space afterwards, the areas will switch places and the happy objects that were sitting on the ‘To’ area (protected against the GC) are now desperatly screaming on the ‘From’ area (back to the risk area), after another minor collection the ‘switch’ happens again and only the chosen ones will be promoted to the Old generation. You must be asking yourself “how many times are these objects dancing between survivor spaces?”, the argument that defines that is the ‘-XX:MaxTenuringThreshold’ (according to what I’ve researched, looks like the default value is 31).

Controlling the spaces

The most obvious JVM arguments are the classic ‘-Xms’ and ‘-Xmx’ (minimum and maximum of heap’s initial space) but to resize the heap in a proporcional way we can use ‘newRatio’ and ‘survivorRatio’, the first one is pretty simple, by default Sun’s JVM (Hotspot) has this parameter defined as ‘2’ (-XX:NewRatio=2), which means, the Old generation has 2 times the size of the Young generation, the argument -XX:NewRatio defines a proportion ‘1:n’ between the Young and the Old spaces.

The argument ‘-XX:SurvivorRatio’ is more interesting, it defines a proportion ‘1:n’ between the Survivor and the Eden space. In other words, if this parameter is configured as ‘-XX:SurvivorRatio=6’ the survivor’s area will occupy 1/8 of the Young space. The Eden occupies 6 spaces and the survivor (from,to) will occupy 2 spaces, hence 6+2=8, here is another drawing:

espacos_ratios

So, the major challenge during a problem analysis or tuning is to find the perfect balance (or the best possible) of parameters that should be passed on to the JVM and, most important, the understanding of the application (how objects behave inside the heap).
