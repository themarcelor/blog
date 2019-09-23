---
layout: post
author: Marcelo Costa
---
Main Garbage Collection algorithms

# The Mark & Sweep algorithm

In cases where a huge clean-up is necessary, this algorithm will stop the processor (the event is known as ‘Stop-the-World’ or ‘Full GC’) and verify each reference to check if the objects can be reached, the ones we can’t ae going to be marked and sweeped later. Even though this thing happens in miliseconds (in normal/heatlhy conditions), it should stop everything, otherwise the garbage that was sitting on ‘0x01abcdef’ can, all of a sudden, show up in ‘0x01fedcba’ during the execution of the next bytecode commands.

Here’s a pseudo-code that will illustrate the algorithm:

```
1	Mark:
2	    add each object in the root set to a queue
3	        for each object x in the queue
4	            mark x reachable
5	            add all object referenced from x to the queue
6	Sweep:
7	    for each object x on the heap
8	        if the object x is not marked, garbage collect it
```

If the GC would not able to work this way, things would be really slow compared to what we have today. Let’s put it like this: “if you have a box of oranges, where there are 2 good ones and 98 of them are rotten”, what would be the best option? Identify and remove the good ones or the bad ones?

![oranges_BOX](https://themarcelor.github.com/blog/assets/img/oranges_BOX.jpg)

I think it would be better to just keep the good ones and get rid of the rest, based on that idea (and the fact that 90% of the objects created during the execution of any application are generally collected) there is another algorithm that we should get familiarized with.

# Generational Copying

This algorithm consist in verifying who survived the last GC (who still holds his reference) and move (copy) to another area, after the surviving object are moved, all that area where they were sitting before is swept away along with the garbage. Isn’t better to take the objects that are going to be used, in a minor population, and throw all the rest away?

But what is the deal with this name? That is because the heap is divided into ‘generations’, the image below shall be impregnated in your brain for the rest of your life:

![java_mem_parts](https://themarcelor.github.com/blog/assets/img/java_mem_parts.jpg)

When an object gets created it is going to the Young Generation, within this space the GC activities happen more often, this allows the short-life objects to be collected more quickly and free-up memory so the application can seize this heap space to continue its execution. If the object survives the GCs that happened on the Young space he will eventually be promoted to the Old generation.
