---
layout: post
author: Marcelo Costa
---
# JVM e Garbage Collection

Oi Pessoal, tudo bem? Starting here the 1º blog post, I’m going to try to create an opuscule (i.e., a wee text) about Garbage Collection e the Java Virtual Machine. This is for those of you that never programmed even a pathetic HelloWorld class and whoever wants to reinforce existing knowledge, e.g., something you learned during the SCJA (Sun Certified Java Associate) studies. Let us jump right into it and start with the first universal truth:

# The JVM is Chaotic

The Garbage Collection is not performed immediatly after an object loses its reference, the JVM waits until a few lost objects start to accumulate. Think of it as a Cleaning Lady that is watching TV and, after her boss’s children finished their fun activities on the living room she decides (whenever she wants) to collect the toys, sweep, vacuum the carpet (i.e., GC).

![stack_and_heap](https://themarcelor.github.com/blog/assets/img/stack_and_heap.jpg)

Look at the image above, on the left side we have the Stack (where the reference variable is) and on the right side we have the Heap (where the actual object is), if we add the following line of code:

`my_birth = null;`

Our cleaning lady will already know that, at some point in time, she will have to take care of that useless object that was left in ‘0x01abcdef’, we can’t do anything about it, just ask nicely:

`System.gc();`

In other words, “Hey Consuela, can you please clean that room over there?” (but that will only make her give more attention to that particular region, she will not do anything immediately).

So, how does the GC works? Well, we know that GC will occur on objects that do not have references, so the first algorithm that we should talk about is the “Mark and Sweep”.
