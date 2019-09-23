---
layout: post
author: Marcelo Costa
---
# A brief comment about the Permanent Generation space (aka, “non-heap”)

This area stores the definition of classes (java.lang.Class) and the Pool of Strings (there’s a cool text about that at the end of this post), which means that, if your application load too many classes into your JVM, you can see the following error: ‘java.lang.OutOfMemoryError: PermGen space’, you can solve that in many ways, one of them is to configure the the argument “-XX:PermSize” to a larger size. In the JVM 1.6 the default size is 8mb to the JVM in client-mode and 16mb to server-mode, and the max size (-XX:MaxPermSize) to both of them is 64mb.

# Pool of Strings (by Kathy Sierra)

>One of the key goals of any good programming language is to make efficient use of memory. As applications grow, it’s very common for String literals to occupy large amounts of a program’s memory, and there is often a lot of redundancy within the universe of String literals for a program. To make Java more memory efficient, the JVM sets aside a special area of memory called the “String constant pool.” When the compiler encounters a String literal, it checks the pool to see if an identical String already exists. If a match is found, the reference to the new literal is directed to the existing String, and no new String literal object is created. (The existing String simply has an additional reference.) Now we can start to see why making String objects immutable is such a good idea. If several reference variables refer to the same String without even knowing it, it would be very bad if any of them could change the String’s value.
> You might say, “Well that’s all well and good, but what if someone overrides the String class functionality; couldn’t that cause problems in the pool?” That’s one of the main reasons that the String class is marked final. Nobody can override the behaviors of any of the String methods, so you can rest assured that the String objects you are counting on to be immutable will, in fact, be immutable.
