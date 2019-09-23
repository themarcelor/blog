---
layout: post
author: Marcelo Costa
---
Hello and welcome to the second part of the “Nashorn and the JVM Monitoring Challenge” series

Hello and welcome to the second part of the “Nashorn and the JVM Monitoring Challenge” series, we will continue our quest to unveil what kind of chaotic things we will see once the JVM starts processing the bytecode that is executing untyped dynamic languages.

Let’s start with some good news: You don’t have to follow the OpenJDK Build instructions to start playing with Nashorn anymore, it was finally integrated to an early build of the Oracle’s JDK 8, I haven’t tried myself (Still playing in Ubuntu 12 with OpenJDK 8) but it should be there.

So, let’s get started, the agenda for today is going to be:

Running a test script with the Nashorn Javascript engine
Understanding Invokedynamic (It should be helpful to dive more into what we are going to see later)
Monitor the JVM while running a Nashorn application
Running a test script
The Nashorn engine can be loaded in a Java class and then, with the instance of the ScriptEngine object, use the eval() method to execute Javascript code. Just write and compile the following class:

```java
import javax.script.*;
public class EvalFile {
 public static void main(String[] args) throws Exception {
     // create a script engine manager
     ScriptEngineManager factory = new ScriptEngineManager();
     // create JavaScript engine
     ScriptEngine engine = factory.getEngineByName("nashorn");
     // evaluate JavaScript code from given file - specified by first argument
     engine.eval(new java.io.FileReader(args[0]));
 }
}
```
Once you have your ‘EvalFile’ class ready, create a dummy .js file just to give it a try, write something like:

`print('Hello World');`

Then you can execute this script like this:

`java -cp nashorn.jar:. EvalFile dummy.js`

In order to speed things up, I’ve created an alias to invoke the ‘jjs’ command-line tool so I don’t have to use this EvalFile class.

`$alias jjs='/var/jdk8/openjdk8/nashorn/bin/jjs'`

now, the .js file can be executed like this:

`jjs dummy.js`

Moving on, my test script here will only be used to explore how we can keep track of the chain of function calls and the number/size of objects that are being allocated into the JVM’s heap, if you really want to experience the power of Nashorn, you can refer to the official ‘Java Scripting Programmer’s Guide‘.

This test comprises two files:

— Model.js —
```js
function Person(name, address, phone) {
    this.name = name;
    this.address = address;
    this.phone = phone;
    this.sayHello = function() {
        return "Hi, my name is " + this.name;
    }
}
```
— testNashorn.js —

```js
#!/usr/bin/jjs
#
var Thread = java.lang.Thread;
load("./Model.js");
print("Welcome to testNashorn.js");
var people = [];
for(var i=0;i<100;i++){
    var p = new Person("Marcelo","Caucaia Street 17","+353 086 5555555");
    people.push(p);
}
//Lots of objects were allocated into memory at this point 
Thread.sleep(30000);
var myFunction = function(){
    var text = people[0].sayHello();
    print(text);
};
myFunction();
```

The first one (Model.js) is where I’m declaring my Person “Class”, 3 attributes and 1 meth.. sorry, function, the second file (testNashorn.js) is the actual program, it first transforms the Thread Java class into a Javascript variable, then loads the Model.js, i.e., runs the Javascript code inside that file and prepares our Person() function/constructor, it declares an empty objects array and enters a loop instruction that is going to create 100 objects in memory, after the ‘for’ loop, as I will need a second to trigger my script to generate a heap dump, I decided to add a ‘Thread.sleep(30000)’ there (30 secs is more than enough), once the program awakes, it declares a function (myFunction) that is going to print the value returned by the ‘sayHello()’ function from the first object stored in the ‘people’ array, this function is then called right afterwards.

Now, we can run the program:

`jjs testNashorn.js`

The output should be something like this:

```
Welcome to testNashorn.js
[30 sec pause]
Hi, my name is Marcelo
```

That’s it, we have our test script, let’s move on to the second topic.

Understanding invokedynamic
Ok, we already know that the Nashorn Javascript engine is 100% compliant with ECMA-262 5.1 and it is fully implemented with the new “invokedynamic” bytecode instruction, therefore, is faster and more compliant than Mozilla’s Rhino, but what’s invokedynamic?

Executing Java code is not the JVM’s solely purpose, every Java code is compiled into bytecode and this is the piece that gets consumed and processed by the JVM, if a programming language can be compiled to bytecode than its instructions can be interpreted by the JVM (e.g., Closure or Scala). The bytecode is an efficient simplified form of non-human-readable code that is executed closer to machine-level instructions, i.e., better performance.

The JVM has approximately 200 “opcodes” to perform invocation of instructions, handle access to fields and control objects and arrays. The following table presents the types of invocation bytecode operations that were available before JDK version 7:

| Opcode | Usage         |

| ------ | ------------- |

| Invokestatic | For static methods |

| Invokevirtual | For non-private instance methods |

| Invokespecial | For private instance |

| Invokeinterface | For the receiver that implements the interface |


A simple invocation to a method starts from a given “Call Site”, which is from where the request is initiated; it is assembled with the name of the method, the signature (access level, modifiers and return type) and the arguments that are processed by this method, the JVM will process this Call Site information and go through a set of operations: It is going to look for that method’s code within memory (Lookup), check if the types involved in the operations match (Type Checking), invokes the actual code (Branch) and then caches the location of that method so, if it is going to be needed again soon, the JVM already knows that memory address and speeds up the process (Cache).

![normalcall](https://themarcelor.github.com/blog/assets/img/normalcall.jpg)

The new “Invokedynamic” bytecode operation allows the JVM to customize how the resources for the Call Site are assembled (dynamically) and also perform a different set of operations within the JVM so the field or method can be accessed (invoked). Instead of the regular Call Site, it integrates bytecode (invokedynamic operation with name and signature) with a bootstrap method, this is the component that will connect the Call Site with the “Method Handle”, once the handle finds the correct way of making this invocation occurs, the JVM will optimize the operation and the invokedynamic bytecode will be attached to the “Target Method” to avoid processing all these steps again. In a scenario where a scripting language that is running within the JVM needs to access a specific function, it is going to initiate the process by providing the bootstrap method with the invokedynamic instructions (name of the function followed by arguments and the return type), the JVM will look for the function within a Method Table (list of functions that are not associated to any object or class) based on the arguments that are defined at runtime (Lookup), once it finds the function, it will perform some language-specific type checking (Type Checking) and then it will finish the bootstrap process connecting the Call Site with the Method Handle so it can be executed (Branch), this connection is performed only once but Call Sites can be connected to new Method Handles.

![dynamiccall](https://themarcelor.github.com/blog/assets/img/dynamiccall.jpg)

If you want a more in depth explanation, I strongly recommend this blog post here:

http://niklasschlimm.blogspot.ie/2012/02/java-7-complete-invokedynamic-example.html

Monitoring the JVM while running a Nashorn application
We have reached the last part today’s post, it’s time to diagnose the Rhinoceros (or… at least, try).

![diagnosenashorn](https://themarcelor.github.com/blog/assets/img/diagnosenashorn.jpg)

Let’s start with some Thread Dumps: if we run our testNashorn.jjs and take a few thread dumps (using the instructions documented in our previous post), this is what we get once we load them into our Thread Dump analyzer:

![screen-shot-2013-08-17-at-17-34-05](https://themarcelor.github.com/blog/assets/img/screen-shot-2013-08-17-at-17-34-05.png)

That’s it, say goodbye to the good-old readable stack trace. On this first analysis we reinforced once more how this change of paradigm will affect the way Java Performance Analysis and Troubleshooting is done nowadays, the chain of execution presented on the stack trace of the “Main” thread resembles bytecode instructions, the best clue to easily identify what initiated each set of instructions is the name of the Javascript file that is declared in the “jdk.nashorn.internal.scripts.Script” class (it can be found at the bottom of the stack trace), there are some familiar things there like the JVM native and internal threads but the rest got pretty cryptic for me.

So, what’s our alternative? As far as I know, there isn’t any. We can only use some arguments to run the program and get some debug data that is supposed to give us some clues as to where the calls are coming from, but it is not very clear. I believe that, if we grok the concepts behind ‘invokedynamic’, we can use the “–print-code” Nashorn argument and produce some debugging output that can be interpreted based on the dynamic calls that are generated by the Nashorn engine:

![screen-shot-2013-08-17-at-18-19-29](https://themarcelor.github.com/blog/assets/img/screen-shot-2013-08-17-at-18-19-29.png)

We can also get more verbose results with the following command:

jjs -Dnashorn.debug=true -Dnashorn.methodhandles.debug.stacktrace=true --log=codegen:info testNashorn.js
What if the program hangs during the execution of a particular function? Under the development phase is pretty easy to just attach a debugger and step through the function calls but what if we are troubleshooting something in the production environment? I’m not sure if we have a valid alternative to that, if you have any ideas, please share on the comment section below.

What about Heap Dumps? Let’s see what we get when we take a Heap Dump during the execution of our testNashorn.js script:

![screen-shot-2013-08-17-at-15-32-46](https://themarcelor.github.com/blog/assets/img/screen-shot-2013-08-17-at-15-32-46.png)

Yep, it also got a little weird here. Since we don’t have packages and typed classes, there’s no way to easily track down where are our “Classes” and the number/size of objects associated to them, I did some investigation and found this “jdk.nashorn.internal.scripts.JO” object that apparently serves as a wrapper to the objects created through Javascript functions, the downside is that it doesn’t separate the objects based on its “Class” (at least I didn’t find any parameter that pointed me anywhere near the “jdk.nashorn.internal.scripts.Script$Person” object), so if you have 100 instances of ‘Person’ and 100 instances of ‘Car’, they will be mingled in this sea of ‘JO’ instances (I haven’t tested other object forms yet, e.g., Object Literals; not sure if we would see something different). So, how do we easily keep track of the size of objects created from a given function()? Well, we could rely on the format of the attributes and play with OQL (Object Query Language) and isolate a given set of objects to determine how much space they are taking up, but that’s just messy. Currently, there are a few DEBUG parameters documented in “$OPENJDK8_HOME/nashorn/docs/DEVELOPER_README“, some of them are quite interesting and might provide the answers we need, e.g.:

print(Debug.map(p));
print(Debug.dumpCounters());
Now, to conclude this post, I leave you with a message from Jim Laskey, this is one of the replies that he sent me when I was questioning his team about these concerns:

“The next round of development will be focusing on tools, so what you are trying to accomplish will get easier. You have an opportunity to provide us guidance on this…. Stack crawls will get better once we start working on debugging APIs.”.

So there you have it, if you are interested in contribute to these debug APIs I hope this post can provide some guidance and/or raise awareness on the difficulties that we might face in a near future where we will be troubleshooting Nashorn-based enterprise applications.

Good Nashorning everyone! 😀
