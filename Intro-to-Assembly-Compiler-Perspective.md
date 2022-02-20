# Introduction to Assembly Language, from a Compiler Perspective

## Structured Control Statements, and how Assembly's if-goto-label can implement them

Popular high level languages use structured control statements; 
these include if-then, if-then-else, for-loop, while-loop, etc..

Assembly language has if-goto-label, for all control structures.&nbsp;
Each high level language control statement has a simple pattern,
and there is an equivalent pattern for each one in assembly's if-goto-label style.

Assembly language patterns use if-goto-label.&nbsp; There are three constructs: unconditional branch, conditional branch, and label.&nbsp;
* Unconditional Branch &#8212; `goto Label;` a jump to another location, no condition is tested.
* Conditional Branch &#8212; `if ( <condition> ) goto Label;` where `<condition>` is any expression that can be evalutated to true or false.
* Label &#8212; `Label:` these labels generate no machine code, they do not execute, and the processor never sees them.

Labels are just marking points in the code with names.&nbsp; At the most primitive level, a label refers to an address, a location, one single point in the code.

Labels do not start a new section; labels are read by the assembler but removed from machine code.&nbsp; 
The processor will happily execute sequential statements regardless of an intervening label.&nbsp; 
When we don't want that, we use branches.&nbsp; Look for the unconditional branch in if-then-else pattern.&nbsp; 

Every application of a pattern for translation requires all new labels.&nbsp; So, I name my labels
based on the control structure along with a unique number for that statement.&nbsp; 
Like `if1`, `endIf1`, `then1`, `else1`, etc..&nbsp; And later `if2`, `endIf2`, `then2`, `else2`.

Let's also note that each pattern's equivalent in if-goto-label is compilable C language syntax.&nbsp; 
This means that you can translate some C code into if-goto-label and compile & run in C before taking to assembly language.&nbsp; 
One could translate one structured control statement at a time, statement-by-statement, keeping the whole program working the whole time.&nbsp; 

(Note some versions of C are strict and won't allow a label without a statement, as that is technically illegal syntax.&nbsp; 
This happens when a label is declared as the last thing in a compound statement, as in `{ ... label: }`.&nbsp; 
A simple solution is to add a null statement using semi colon following the label declaration as in: 
`{ ... label:; }`.)

Once you have removed all the structured control statements from a program, the control flow translates to assembly language rather easily.&nbsp; 
Both unconditional branches and labels translate rather directly to assembly language.&nbsp; While 
conditional branches translate to sequences involving compare followed by branch on condition instructions.&nbsp; 
Some processors like MIPS and RISC V can do limited conditional branches in one instruction.

#### The if-then statement

        if ( <condition> ) 
            <then-part>

    -------- equivalent in if-goto-label -----------------

        if ( ! <condition> ) goto endIf1;
        <then-part>
    endIf1:

Notice that the condition is negated: this is because in if-goto-label we are telling the processor when to skip the then-part
rather that when to execute the then-part.

#### The if-then-else statement

        if ( <condition> ) 
            <then-part>
        else
            <else-part>
            
    -------- equivalent in if-goto-label -----------------

        if ( ! <condition> ) goto else1;
        <then-part>
        goto endIf1;
    else1:
        <else-part>
    endIf1:
   
Again the condition is reversed because we are telling the processor when to skip the then and do the else rather than when to do the then part.

The then-part ends with an uncondtional branch.&nbsp; If that code is executing we know that the if statement's condition is true, and the then-part has fired.&nbsp; 
Upon completion of the then-part the next statement after the entire if should execute, certainly not the else-part.&nbsp; The `else1:` label does not break
the processor's execution, only instructions can do that, hence the unconditional branch, the goto.

#### The while-loop statement

        while ( <condition> ) 
            <loop-body>
            
    -------- equivalent in if-goto-label -----------------

    loop1:
        if ( ! <condition> ) goto endLoop1;
        <loop-body>
        goto loop1;
    endLoop11:

Once again, the condition is reversed because we are telling the processor when to exit the loop whereas the C form says when to stay in the loop.

#### The for-loop statement

I translate for-loops into while and then use the while-loop pattern.

        for ( <initializers>; <condition>; <incrementings> ) 
            <loop-body>
            
    -------- equivalent as while-loop  -----------------

        <initializers>
        while ( <condition> ) {
            <loop-body>
            <incrementings>
        }

### Translating Composed Structured Statements

One big advantage of structured statements is how easily they compose.&nbsp;
One statement can be nested within another.&nbsp; One for loop nested within another.&nbsp; 
An if statement nested within a for loop.&nbsp; 

This nesting does not affect the pattern translation: an inner statement can be translated to if-goto-label
before or after an outer statement.&nbsp; You can work from the outside in or inside out.&nbsp;
There is a third approach, which is to work more sequentially from start to finish but that means keeping several pattern
translations in flight, so I don't recommend it.

`<example>`

### Patterns in if-goto-label often end with Label

Consider the following sequence of two statements, A & B where B follows A.

       if ( i == count-1 )                  // Statement A
           print("then-part")               //     (this is nested within the Statement A)
       print("next");                       // Statement B

    -------- equivalent in if-goto-label -----------------

       if ( i != count - 1 ) goto endIf1;    // Statement A
       print("then-part");                   //     (nested within Statement A)
    endIf1:                                  // end of Statement A
       print("next");                        // Statement B

In high level language successive statements execute one after another until the end of a function.&nbsp; 
When a next statement, B, follows a prior, A, that next statement should run when the prior is completed.&nbsp; 
This is true whether the prior statement is an if-then or a while-loop, either way, when A is completed
the B is what runs next.&nbsp; Thus, whether the if-then fires (is true so runs the then-part)
or not, the next statement after should run.&nbsp; 
When an if-goto-label pattern ends with a label, that is facilitating execution of the next sequential statement.

In assembly programming, many people like to choose label names based on what's coming up next, what's in that next statement, B, for example, 
maybe naming the label `PrintNext:`.&nbsp;
However, I choose label based on the structured control statement whose pattern equivalent that produces the label(s).&nbsp; 
Here the if-goto-label pattern equivalent is from an if-then statement, so the label is endIf1: .&nbsp; 
If you write a compiler, you'll find it will naturally want to use a similar approach 
in that the labels are associated with the patterns being translated rather than the next coming after them.

### Indentation

In if-goto-label, I use flat indentation.&nbsp; All statements at 1 level of indentation, all labels at none.&nbsp; 
There are other approaches using indentation to reflect nested structured control statements.&nbsp; 
But flat is a traditional approach to assembly language.

### Simplification of Short Circuit Operators
  
The operators && along with || provide for compound conditions.&nbsp; These need to be simplified for complete if-goto-label.&nbsp; 
However, they simplify easily in if-goto-label.

The idea is two split the two operands (left and right) of `&&` into two statements; same with `||`, to split the left and right operands into their own if-goto-label statements.

#### Conjunction, `&&`

With &&, if first condition is false, we skip the then-part.&nbsp; Only if the first condition is true do we even test the second condition, and then,
if the second condition is false, we also skip the then-part.&nbsp; So, the then-part only runs if both conditions are true.

        if ( a && b )
            <then-part>

    -------- equivalent in if-goto-label -------- split left & right operands of && into their own if-goto
    --------------------------------------------- reversing both conditions
    
        if ( ! a ) goto endIf1;                   // if the first condition is false, skip the then-part
        if ( ! b ) goto endIf1;                   // if the second condition is false, skip the then-part
        <then-part>
    endIf1:

This is now in assembly-friendly full if-goto-label form.

#### Disjunction, `||`

With ||, if the first condition is true, we jump ahead to fire the then-part.&nbsp; Only if the first condition is false, do we test the second condition, and then,
if the second condition is false, we skip the then-part.&nbsp; So, the then-part runs if either condition is true.&nbsp; 
An additional label is needed for to decompose this construct, which allows skipping over the second condition directly to the then-part.

        if ( a || b )
            <then-part>

    -------- equivalent in if-goto-label -------- split left & right operands of || into their own if-goto, 
    --------------------------------------------- keeping 1st condition forwards but reversing 2nd condition
    
        if ( a ) goto then1;                      // if first condition is true, run the then-part
        if ( ! b ) goto endIf1;                   // if second condition is false, skip the then-part
    then1:
        <then-part>
    endIf1:
