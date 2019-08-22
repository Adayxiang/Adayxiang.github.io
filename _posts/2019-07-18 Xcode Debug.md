---

Layout: post

Title: Xcode Debug Instruction

Subtitle: Using Xcode's various debugging bug methods in iOS development

Date: 2019-07-18

Author: adayxiang

Header-img: img/post-bg-ios9-web.jpg

Catalog: true

Tags:

    - iOS

    - Development skills

    - Debug

---

 

 

#前言

 

>BUG, simply speaking, the result of the program is different from the expected one. Let's talk about the DEBUG method in Xcode .

>

>[ Reference blog post ](http://www.cnblogs.com/daiweilai/p/4421340.html#quanjuduandian)

 

 

 

#断点Debug

 

- ordinary breakpoints

- global breakpoint

- conditional breakpoint

 

#### 1. Ordinary breakpoints

Look at the picture

 

![](http://ww4.sinaimg.cn/large/65e4f1e6gw1f8rti38wlxj20ke0d3n0h.jpg)

 

Stop when the program runs to the breakpoint, then step through the steps

![](http://images.cnitblog.com/blog2015/680363/201504/131002381048966.png)

 

 

#### 2. Global breakpoints

 

When the program is running crashes, it will automatically break to appear crash of lines of code

 

![](http://images.cnitblog.com/blog2015/680363/201504/130933043392329.png)

 

#### 3. Conditional breakpoints

 

 

If we use a breakpoint in a loop, if the loop is executed 1 million times, then your breakpoint should be executed so many times, don't you think that the egg is cold and sad? So we do this:

 

Edit breakpoint

 

![](http://ww1.sinaimg.cn/large/65e4f1e6gw1f8rw64yys0j207i03laah.jpg)

 

Add Condition Condition

 

![](http://ww2.sinaimg.cn/large/65e4f1e6gw1f8rw52q1tjj20ct04lmxo.jpg)

 

 

![](http://ww3.sinaimg.cn/large/65e4f1e6gw1f8rw44p4ykj20ln0g10vg.jpg)

 

You can also execute an event in the Action when the conditional breakpoint is triggered.

 

![](http://ww3.sinaimg.cn/large/65e4f1e6gw1f8rwq16872j20cv07amyg.jpg)

 

Such as: output information

 

![](http://ww2.sinaimg.cn/large/65e4f1e6gw1f8rwms50t3j20dj07bjso.jpg)

 

#### 4. Method breakpoint

 

# Print debugging ( NSLog )

 

Although ARC has made memory management simple, time-saving, and efficient, it is still important to track some important events in the life-cycles of an object . After all, ARC does not completely rule out the possibility of memory leaks, or trying To access a release object.

 

- NSLog

 

Strengthen NSLog

 

```

//A better version of NSLog

#define NSLog(format, ...) do { \

Fprintf(stderr, "<%s : %d> %s\n", \

[[[NSString stringWithUTF8String:__FILE__] lastPathComponent] UTF8String], \

__LINE__, __func__); \

(NSLog)((format), ##__VA_ARGS__);

Fprintf(stderr, "-------\n"); \

} while (0)

```

Console output

 

```

<ViewController.m : 32> -[ViewController viewDidLoad]

2016-10-14 17:33:31.022 DEUBG[12852:1238167] Hello World!

-------

```

 

Use NSString to output multiple types

 

![](http://ww4.sinaimg.cn/large/65e4f1e6gw1f8rxvn6fqlj20nc05cgoh.jpg)

 

- Turn on zombie objects

 

Xcode can put those already release out objects have become " zombie " , when we visit a Zombie when the object, Xcode can tell us the object being accessed is one that should not exist in the target. Because Xcode knows what this object is, it Let us know where the object is and when it happened.

So Zombies is your good friend! He can make the information you output more specific!

 

Specifically do this: (Zombies can only be used in the simulator and OC language )

 

![](http://images.cnitblog.com/blog2015/680363/201504/130941016986159.png)

 

# Console (lldb command )

 

 

 

LLDB is an open source debugger with REPL features and C++, Python plugins. The LLDB is bound inside Xcode and exists in the console at the bottom of the main window. The debugger allows you to pause the program at a specific time. You can view The value of the variable, execute the custom instructions, and follow the steps you think are appropriate to manipulate the progress of the program. (The is a general explanation of how the debugger works.

 

You may have used the debugger before, even if you just add some breakpoints to the Xcode interface. But with a few tricks, you can do something very cool. The GDB to LLDB reference is an excellent overview of the commands available for the debugger. You can also install Chisel , which is an open source LLDB plugin compilation that makes debugging more interesting.

 

Reference:

 

[ Walk with the debugger - waltz of LLDB ] (http://objccn.io/issue-19-2/)

 

[ Preliminary study of LLDB debugging commands ] (http://www.starfelix.com/blog/2014/03/17/lldbdiao-shi-ming-ling-chu-tan/)

 

[About LLDB and Xcode] (https://developer.apple.com/library/content/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/Introduction.html)

 

[The LLDB Debugger] (http://lldb.llvm.org/tutorial.html)

 

#### Foundation

###### *help*

Enter `help` at the console to display the lldb command supported by the console.

 

###### *print*

Print value

 

Abbreviation `p`

 

Print is an abbreviation for `expression --`

 

 

![](http://ww4.sinaimg.cn/large/006y8lVagw1f8vakv88vuj30b204s74x.jPg)

 

Printk can print in the specified format

Such as

` Default p`

 

` hexadecimal p/x` ,

 

` binary p/t`

 

```

(lldb) p 16

16

 

(lldb) p/x 16

0x10

 

(lldb) p/t 16

0b00000000000000000000000000010000

 

(lldb) p/t (char)16

0b00010000

 

```

You can also use p/c to print characters, or p/s to print ACRSII with a null-terminated string p/d ( Translator's Note: a string ending with '\0' ) .

 

Complete list [ Click to view ] (https://sourceware.org/gdb/onlinedocs/gdb/Output-Formats.html)

 

 

###### *po*

 

Print object, which is an abbreviation of `e -o --`

 

###### *expression*

 

#### Process Control

 

When you insert a breakpoint through the side slot of Xcode 's source editor ( or by the following method ) , the program will stop running when it reaches the breakpoint.

 

There are four buttons on the debug bar that you can use to control the execution flow of the program.

 

![](https://objccn.io/images/issues/issue-19/Image_2014-11-22_at_10.37.45_AM.png)

 

From left to right, the four buttons are: continue , step over , step into , step out .

 

The first, the continue button, cancels the program's pause, allowing the program to execute normally (either until until continues, or to the next breakpoint ) . In LLDB , you can use the process continue command to achieve the same effect, its alias is Continue , or it can be abbreviated as c .

 

The second, step over button, executes a line of code in a black box. If the line of code is a function call, then it will not jump into the function, but will execute the function and then continue. LLDB can use thread step -over , next , or n commands.

 

If you really want to jump into a function call to debug or check the execution of the program, use the third button, step in , or use the thread step in , step , or s command in LLDB . Note that the next and step effects Are the same when the current line is not a function call .

 

Most people know c , n and s , but there is actually a fourth button, step out . If you have accidentally jumped into a function, but you actually want to skip it, the common reaction is to run n repeatedly until the function returns. In fact, in this case, the step out button is your savior. It will continue to the next return statement (unthe end of a stack frame ) and then stop again.

 

###### frame info

 

Will tell you the current number of lines and source files

 

```

(lldb) frame info

Frame #0: 0x000000010a53bcd4 DebuggerDance`main + 68 at main.m:17

 

```

 

###### Thread Return

When debugging, there is a great function that can be used to control the program flow: thread return . It has an optional parameter that, when executed, loads the optional parameters into the return register and then immediately executing the return command to jump out of The current cause of the ARC reference count or invalidate the cleanup part of the function. But performing this command at the beginning of the function is a very good Way to isolate this function and fake the return value .

 

`(lldb) thread return NO`

 

#### Without breakpoint debugging

 

The program is running, click on the pause button , to enter debug state, can do to a global variable

 

![](http://ww4.sinaimg.cn/large/006y8lVagw1f8vd4vy66ej307300xjr8.jpg)

 

 

#工具调试(instruments)

Instruments Xcode comes with a lot of tools for everyone to use, open as follows:

 

![](http://ww1.sinaimg.cn/large/006y8lVagw1f8ve05g45cj30qd0f276o.jpg)

 

**leaks** Memory Leak Checker

 

![](http://ww4.sinaimg.cn/large/006y8lVagw1f8ve5wnnr6j30li0c1wgd.jpg)

 

View after running

 

![](http://ww4.sinaimg.cn/large/006y8lVagw1f8vebiu6r5j30se0kdqcr.jpg)

 

# View debugging

 

Enable debugging views : Run app , pressing the bottom of the Debug View Hierarchy button, or choose from the menu Debug> View Debugging> Capture View Hierarchy to start debugging views.

 

![](http://ww1.sinaimg.cn/large/006y8lVagw1f8vejy3rmgj30by01kmx8.jpg)

 

After launching the view debug, Xcode takes a snapshot of the application's view hierarchy and presents a 3D prototype view to explore the hierarchy of the user interface. In addition to showing the view hierarchy of the app , the 3D view shows the position, order and View size of each view, and how the portrait interact.

 

#模拟器Debug

 

Compile and run the application, select the emulator, and select the Color Blended Layers option from the Debug menu .

 

![](http://ww2.sinaimg.cn/large/006y8lVagw1f8vezdqlh1j3092075dgz.jpg)

 

You will then see that the app 's user interface is overlaid with red and green, showing which layers can be overlaid and which layers are transparent. Mixed layers are computationally intensive, so it is recommended to use opaque layers served possible.

 

![](http://ww3.sinaimg.cn/large/006y8lVagw1f8vf07u522j30ag0j1q36.jpg)

 

#结语

The debugging methods currently known are pro