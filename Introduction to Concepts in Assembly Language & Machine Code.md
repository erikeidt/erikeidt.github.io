There are several aspects to learning assembly language, which is the human readable version of the machine code of the processor.

---

* Basically other languages are at a logical level, whereas machine code is very much at a physical level.

At the logical level we have variables and code that manipulates them &#8212; generally speaking whenever we want we can declare/define a new variable, as many and as often as we like.&nbsp; We have notions of scope and limited lifetime, so variables come and go dynamically.&nbsp; Further, in a language like C or Java, variables are declared with a type, and whenever you reference the variable, the language remembers the declared type.

By contrast, machine code is much more at a physical level: there are always fixed number of CPU registers and memory, in the case of LC-3 there are 8 registers and 64k of memory.&nbsp; The registers and memory simply remember the last value the program in them.&nbsp; They are sized (# of bits) but otherwise untyped (by comparison with C or Java).&nbsp; Further, there are no variable declarations in machine code &#8212; that is, none that the processor reads and remembers.&nbsp; 

*Assembly language allows defining labels, for both code and data, but to be clear the processor never sees them.&nbsp; So, each instruction that uses physical resources tells the processor what it needs to know about the current type of that resource &#8212; such as whether signed or unsigned, or in the case of memory, the size of the variable &#8212; let's note here that LC-3 is simplified as it doesn't differentiate between signed vs. unsigned, or byte vs. word.&nbsp; Various assemblers across different systems go to different lengths, from none (most) to a bit, to try to issue compile time errors on misuse of data declarations.*)

So, when we write assembly language, we translate our pseudo code: logical code with lots of typed variables of limited lifetime, in part by mapping logical variables onto the fixed physical resources.&nbsp; There are often more variables than CPU registers, especially when some of the registers have dedicated purpose, like stack or return address.

The physical resources need serious management.&nbsp; They are often repurposed &#8212; this is done by simply writing a new value, and then it is up the compiler or programmer to make sure that the physical resources implement the logical variables for their lifetimes.

---

 * Today's other languages generally employ structured programming, whereas in assembly language/machine code we have if-goto-label.

Structured programming has structured control statements &#8212; that nest nicely.&nbsp; These control statements include if-then, if-then-else, while-loop, for-loop, do-while.&nbsp; If you want to nest one for-loop within another just use those {}'s or indent as appropriate, but it is quite simple.

Assembly language generally has only if-goto, goto, and labels for control flow.&nbsp; Using these, we can mimic any control flow from structured languages, as well as a myriad of non-standard (ie: confusing) control flows as well as mistaken & accidental forms.

All structured statements have translation in if-goto-label.&nbsp; Each translation is a transformation of a pattern in structured form to a pattern in if-goto-label form.&nbsp; Follow the patterns properly and you'll reproduce the control flow of your pseudo code &#8212; very easy to take short cuts here and make confusing mistakes, so I encourage a methodical approach here.

With nested control structures, when following the patterns, we can translate the patterns of one logical statement at a time: for example, we can do the inner loop first leaving the outer loop for next, or vice versa.&nbsp; Just as we can nest nicely with structured statements, we can do so in assembly &#8212; though it is more error prone and we don't get good error messages from the assembler.

Let's also note here that both structured patterns and if-goto-label patterns can be expressed in C &#*8212; and seeing these transformations from C to C, makes them easy to visualize.

---

 * Other languages have rich expressions: having operators of many levels of precedence, and as complex as you like using `()`'s.

Rich expressions require thought to translate to assembly language.&nbsp; One step in the process could be three address code, which still uses infinite variables (logical variables), but breaks down complex expressions into simple statements.&nbsp; See https://en.wikipedia.org/wiki/Three-address_code .&nbsp; You can see that TAC introduces variable of intermediate or short-term use, these are generally called temporaries.&nbsp; Even simple expressions translated to assembly need temporaries, and those temporaries as logical variables need to be mapped to physical resources, just like the longer-lived variables.

---

 * Function calls, stack frames, parameter passing, return values, hidden parameters, global data structures

Function calls are one of the most complex subjects for assembly language programming, especially on register rich machines.&nbsp; As you recall from above, the CPU registers need to be constantly repurposed.&nbsp; Imagine sharing 8 (on some machines) or 32 (on others) among thousands of functions.

In order to facilitate this, by software convention, we subdivide the physical resources &#8212; the CPU registers &#8212; into partition sets.&nbsp; One such set has "call preserved" vs. "call clobbered" (aka "scratch") registers.&nbsp; The former set is designed for a usage scenario that involves variables that are live across a function call, whereas the latter involves variables that are not.

In order to understand which registers to use (or even when to use memory) we need an analysis of the pseudo code being translated, in terms of the concept of live-ness.&nbsp; Liveness is the property that refers to the range or stretch of code across which a variable is live.&nbsp; This range begins when a variable is set or update, and ends after the last usage of the variable.&nbsp; Variables used in loops need to take the loop into account.&nbsp; For example the iteration control variable (e.g. i in a for i = loop), is tested and incremented in the loop, and thus is live for the duration of the loop.

Regarding liveness: when two variables have overlapping liveness, they cannot both occupy the same physical resource (e.g. CPU register).&nbsp; When there is function calling, some registers are necessarily repurposed (parameter registers, return registers), while others are potentially repurposed (scratch registers).&nbsp; Function calls can be thought of as a single instruction that potentially touches all the scratch registers, and therefore wipes them out.&nbsp; We should not leave our live variables in these registers across a call.

See this article: https://stackoverflow.com/a/64846929/471129


In assembly language there are two concepts hidden from C programmers.

* One is the return address

* Another is the stack pointer.
