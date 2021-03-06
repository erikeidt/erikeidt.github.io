 * Temptations to resist

* Use pseudo code, I strongly recommend C, and testing the C to make sure it actually works.&nbsp; It is no fun debugging a design flaw in assembly language, and, a small change in the pseudo code (to fix a bug) can result in a large change to assembly code.&nbsp; It is so much easier to find & fix design flaws in C than in assembly, and if you make changes to the C code after doing the assembly translation, it can be hard to get the old assembly to reflect those changes.

I see many tempted to:

1. not use working pseudo<br/>
  * thinking the problem is simple and doesn't need it (even for simple things we often don't have sufficient clarity to take that to assembly language when we're first leaning assembly), 
  * thinking that it will slow you down (it won't, it will speed you up), 

2. Making algorithmic changes when translating into assembly language, such as 
  * converting an algorithm from arrays and indexing variables to using pointers variables instead
  * converting a while loop into a do while loop

Do these transformations in C and make sure they work, instead of during translation to assembly

---

* I recommend using logical transformations to take your pseudo code into assembly language.&nbsp; Small steps repeated as needed.

* Be proficient at single step debugging &#8212; learn it in your other languages first, and do it in assembly language.

* Don't write a ton of code before assembling it &#8212; write small sections, assemble, run, test & debug them.

