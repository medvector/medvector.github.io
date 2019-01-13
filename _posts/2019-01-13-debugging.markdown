---
layout: single
title:  "3 small tips for successful debugging"
date:   2019-01-13
categories: programming
---
They say it sucks that actual software developers spend most of their working day
fixing bugs, not developing new exciting features.
Well, I personally love bugs. Especially the most sophisticated ones. Bugs are like
detective stories, where you have small clues and you have to figure out who the murderer is.

Btw, I attended a [good talk](https://youtu.be/HuuYwUxM-ZY) on the subject of loving your bugs in PyCon 
and I'd like to recommend it.

As detectives have their devices, programmers have debuggers. Debugging itself is an art
and you have to perfect it.
Let me offer you my 3 small tips for successful debugging.

**1. Minimize steps to reproduce the problem**

Reproducing the problem before trying to fix it is a must-have. After all, that's the
first tip every single article on the internet about debugging mentions. 

*Minimizing* steps, on the other hand, is not what all the developers do.
Probably because it is an extra effort which often seems unnecessary. However, it can save
you so much time. Nothing is perfect and fixing this particular bug might be trickier than
you thought at the beginning and by the point you realized it you already had repeated 5 actions
instead of only 2 for the fiftieth time. Your fix might have been incorrect and when QA returned the ticket 
into the development phase you literally feel pain looking at the steps to reproduce. *Oh no, not this again...*
There are bugs caused by race conditions which only reproduce 1 time out of 10, so you find yourself
doing the same stuff all over again hoping that this time the problem occurs.

This trick can take many forms.

I personally hate doing repetitive actions and ideally, I prefer somebody or something else doing them for me. For example, I'd always ask myself if I can *write a test* quickly and fix it
instead of clicking things in the real application.

It also makes sense to minimize your setup. If the bug occurs when you execute some SQL query, try
reproducing it requesting not the whole huge table, but only 10 entries.

In the example above about race conditions, you could tweak your code so as race condition actually
occurs every time: add some barriers or silly enough just add several sleep statements in the right places.

Sometimes it is about pro usage of the debugger: conditional and exception breakpoints. Why iterate
thought the whole loop if you can just stop at a breakpoint only if the value in question is null?

I confess, I often neglect this advice and always to no good. For example, last week I was fixing a bug.
Because of another bug when the first bug happened I had to restart my application to reproduce it one
more time. I'm sorry to say that I did it more than one or even five times before I finally fixed the
second bug so as the first bug would be reproducible without the application restart.

**2. Explain each line out loud**

My friends with competitive programming background taught me this very simple trick. And this is pure
black magic.
If you're stuck and you absolutely can't figure out what's wrong with your code,
just start explaining to yourself out loud what each line exactly does. 
At some point during this process, a viable hypothesis about the bug just magically comes to you.

Actually magic doesn't exists (what a bummer!) and this trick works because it allows you to view your code
in a very focused mode. In this mode you're much more likely to notice typos: for example, you called <code>go_back()</code>
function instead of <code>go_forward()</code>. You're also much more likely to notice stuff that you don't
understand. And your lack of understand is what caused the problem in the first place. And, wait, this function
isn't supposed to be called in this environment! Let me fix it...

**3. The best debugging is no debugging**

I once read that if it comes to debugging you should consider it a failure.
Well, I do agree that it's great if it's possible to fix the problem without
executing your code step-by-step. 
What instruments can help you archive it? 

The first instrument is *assertions*. Assertions are the way to automatically check your assumptions
about your code. And a lot of bugs are caused by wrong assumptions.

Assertions also come with several additional perks. They allow spotting the source of the problem
even if its symptoms appeared later on the execution stack. Assertions also help other 
people understand what data can be passed to functions because all the assumptions are there in the code 
(Yeap, it's good when a type system allows to do it, but it's often not enough and sometimes it's even 
not possible to modify the function's signature).
What I like the most, assertions help to understand what happened without any effort from
the end user.

In IntelliJ IDEA assertions are used, for example, to ensure that you call particular
functions only on UI thread. And these assertions help a lot because they tell you
exactly what you did wrong.
 
The second instrument is *logging*. It's not much to tell here. Logs help you to understand
what your application did without executing it step-by-step.

One thing that irritates me though is when people log stuff without throwing an exception
or asserting.

For example, various <code>log("a is null")</code> messages usually mean that IDE inspections
highlighted possible NullPointerException and a developer just did something like this to shut it up:

{% highlight java %}
if (a == null) {
  log("a is null");
  return;
}
{% endhighlight %}

So what can I do when I see this error message in logs? I don't have stack trace and I can't understand why a being null is so bad.
So I have to search for this message in the code base and actually read the code to understand what operation we couldn't perform because 
of being null.

On the other hand, if we'd replaced the whole piece of code above with single <code>assert a != null;</code>
we'd got stacktrace showing us were we failed and also allowing us to navigate back to the code in an IDE.

*Do you also know any tools to spot bugs without debugging? 
Or do you have nice tips on how to debug your code?*







