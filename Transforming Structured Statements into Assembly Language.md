# Transforming Structured Statements into Assembly Language

### Conditional Expressions in the If-Goto Style of Assembly Language

[[TOC]]

## Introduction &#8212; Structured Statements vs. If-Goto

High level languages offer structured statements: constructs such as if-then-else, while-do, do-while, repeat-until, and for-loop.&nbsp; A particularly useful feature of structured statements is that they can nest.&nbsp; A structured programming language is one that supports structured statements.

In general, assembly language does not offer structured statements.&nbsp; Instead it offers three constructs:

1. statement labels &#8212; these are used to uniquely identify places in code 
2. a "goto label" statement &#8212; also called an unconditional branch, and, 
3. an "if condition then goto label" statement &#8212; also called a conditional branch.

We can call using just these for conditional logic as an "if-goto" style of programming.&nbsp; All structured statement constructs can be translated into the if-goto style, and, whether this is done explicitly or just as a mental exercise, this transformation is a first step in accomplishing (the equivalent of) structured statements in assembly language.

### Transformation of Structured Statements into the If-Goto Style

Some high level languages like the "C" programming language also offer if-goto, goto, and labels, which means we can transform structured statements in "C" into the if-goto style in "C".&nbsp; So, let's take a look at this!

### The if-goto Construct

First, let's talk about the if-goto construct; it looks like this:

        if ( condition ) goto some_label;	// conditionally goto to some_label
        // otherwise fall through to here
        
        ...
        
    some_label:

We call this if-goto construct a *conditional branch*: during execution of this statement, if the test condition evaluates to true, the goto will fire (execute) and alter the (normally sequential) flow of control, causing the statement at "some_label" to be the next to execute (and also also causing the succeeding code to be skipped).

On the other hand, if the test condition evaluates to false, the goto will not fire (will not execute) and the flow of control will remain as normal sequential flow: the conditional branch will be not taken and control instead will *fall through* to the following statement.&nbsp; So, the conditional branch statement will either cause the branch to be taken or will fall through.

A note on assembly language: typicial assembly language programming style uses labels and statements (instructions) as follows: labels typically appear at the beginning of the line (often on their own line), and all other statements are indented one tab stop.&nbsp; In showing code using if-goto, even though in "C", I'll use this assembly language programming style.

### The Structured If-Then Statement

With the nature of the if-goto conditional branch construct in mind, let's transform the simplest structured if-statement, the if-then (without an else).

    if ( condition ) {
        then-part
    }

The transformation is fairly straightforward :

        if ( ! condition ) goto nextStatement;
        then-part
    nextStatement:

We introduce a label (`nextStatement:`) that is placed beyond the then-part.&nbsp; We use this new label in an if-goto statement, which placed before the then-part.&nbsp; The then-part is either executed or skipped.

We need the if-goto statement to skip around the then-part under the right conditions.&nbsp; The following table helps explain why we must negate (reverse) the condition in the transformation.

| Original Condition in Structured If | Desired Action | If-Goto Usage |
|:-:|:-:|:-:|
| condition is true | execute the then-part | fall through so then-part executes &#8212; i.e. don't fire the goto |
| condition is false | skip the then-part | fire the "goto" that skips the then-part |

Thus, we negate the original condition being evaluted in the if-goto statement, so that the branch is taken when the original condition is false.

### Out of Line Transformation

There is another way to translate a simple if-then statment into the if-goto style:

        if ( condition ) goto thenPart;
    nextStatement:
    
        ...

    thenPart:
        then-part
        goto nextStatement;

This transformation places the then-part out of line with the condition test.&nbsp; Under the right circumstances, the then-part could actually be oriented within the source file either before or after the condition test!&nbsp; An out of line transformation like this is unusual, though is sometimes seen and does work.&nbsp; For the rest of this article, though, we will not discuss this further and instead concentrate on the well-known patterns of transformation, which are preserving orientation and swapping orientation of some elements of the structured statment &#8212; this is discussed next.

### Transformation Orientation

Transformation orientation is a term that I use that goes to the ordering of the subcomponents of structured statements.&nbsp; For example, a structured if-then-else-statement has the following subcomponents: a condition, a then-part, and an else-part; whereas a while statement has a condition, and a loop body.

We have two choices for the orientation in the common patterns of if-goto transformation: 

1) We can keep the relative orientation of the original elements of the structured code &#8212; this means that we will see expressions and statements in the *same order* in the transformation as we see in the original structured statement.&nbsp; The advantage of this approach is a clear(er) correspondence between the original structured code and the if-goto style transformation, or,

2) We can deviate from the original ordering &#8212; which will subtly change the assembly language code used, can sometimes offer efficiency improvements, though at the expense of poor correspondence with the original structured code (this is unimportant in some environments).

The subsequent sections will present both approaches for various statements, and discuss respective merits.&nbsp; Often the choice doesn't matter, we just have to pick one.&nbsp; We can pick different orientations no matter whether structured statements are nested or sequential &#8212; as long as we apply one and only one to each structured statement.

## Transforming the Structured If Statement into If-Goto Style

### The Structured If-Then-Else Statement

Let's look at the construct of the structured if-then-else statement:

    if ( condition ) {
       then-part
    } else {
       else-part
    }

There are several approaches to transforming this into the if-goto style.&nbsp; In what I'll call the first approach, we *keep the same ordering* of statement elements as found in the structured form.

#### The Structured If-Then-Else Statement Transformed: Original Orientation

We want to branch to the else-part (and skip then-part) when the original condition is false.&nbsp; This is done by reversing the condition so we will take a branch around the then-part and to the else-part when the original condition is false.

We also need to avoid executing both the then-part and the else-part in any given execution of the (transformed) if statement.&nbsp; This is done with an unconditional branch placed at the end of the then-part that skips the else-part.&nbsp; It is unconditional since whenever we are actually executing the then-part, we always want to skip the else-part.&nbsp; Thus, fully formed, we have the following transformation:

        if ( ! condition ) goto elseLabel;
        then-part
        goto nextStatement;
    elseLabel:
        else-part
    nextStatement:

#### The Structured If-Then-Else Statement Transformed: Reverse Orientation

In the above, when we keep the same ordering we must test the reverse condition.&nbsp;In an alternative shown next, we *change the ordering of some elements **instead of reversing the condition***.&nbsp; Here, the relative orientation of else-part and then-part are swapped from that in the structured form.

        if condition goto thenLabel;
        else-part
        goto nextStatement;
    thenLabel:
        then-part
    nextStatement:

In this form, when the condition evaluates to true, we take the branch to the then-part (which is here located/oriented after the else part!).&nbsp; And when the condition evalutes to false, we fall through into the else part.

Either approach to orientation is viable.&nbsp; To be clear, only one should be used as we want exactly one reversal &#8212; not zero and not two!

## Structured Loop Statements

### Transforming The While-Do Loop into If-Goto Style

Let's look at a simple while loop:

    while ( condition ) {
        body
    }

#### The While-Do Loop Transformed: Original Orientation

With the while loop, we do not have an else part, just the condition test and the body.&nbsp; In preserving orientation, transformation will place the condition test first, and the body second:

    loop:
        if ( ! condition ) goto theLoopIsDone;
        body
        goto loop;
    theLoopIsDone:

As you can see the conditional branch at the top of the loop uses a taken branch to exit the loop, and falls through to continue the loop for another iteration.&nbsp; As exiting the loop is the opposite of the intent of the stated condition in a while statement, we reverse the condition.

#### While-Do Loop Transformed: Reverse Orientation

We also have an alternative using reversed orientation:

        goto testTheCondition;
    loop:
        body
    testTheCondition:
        if ( condition ) goto loop;

Here the body and the condition are oriented in reverse of the structured form.&nbsp; Since the condition test is now at the end of the loop, it should *take* the branch to continue the loop when the condition is true (rather than taking the branch to exit).&nbsp; The reordering has reversed the sense of the taken branch (continue the loop rather than exit the loop), so we now test the unreversed original condition.

A `nextStatement:` label is not needed since the next statement will be reached by falling through on repositioned if-goto.&nbsp; (A `break;` statement in the loop body to exit the loop would require restoring the `nextStatement:` label.)

There is one more effect of this orientation &#8212; namely that the number of instructions inside the loop is smaller by one: the unconditional goto statement (here at the top) is no longer a proper part of the loop &#8212; it fires only one time at the beginning of the loop.&nbsp; By comparison, the goto in the original orientation fires each time through the loop, adding one instruction to each iteration (though technically saving one instruction if the loop has no iterations).&nbsp; This is a common compiler-performed optimization, and should be expected in optimized code.

### Transforming the Repeat-Until Loop into If-Goto Style

The structured repeat-until statement:

    repeat
        body
    until ( condition );

The transformation pattern is very similar to the while-do loop in reverse orientation with the difference that here the nature of the condition is to exit the loop when the condition is met, so the condition is reversed.

    loop:
        body
        if ( ! condition ) goto loop;

This transformation is in original orientation, and, a reverse orientation doesn't make too much sense here so won't be shown.

### Transformaing the Do-While Loop into If-Goto Style

The do-while is the same as the repeat-until, except the sence of the condition is reversed between these structured statements.

The structured repeat-until statement:

    do
        body
    while ( condition );

The transformation pattern is very similar to the repeat-until loop though with the condition unreversed.

    loop:
        body
        if ( condition ) goto loop;

This transformation is in original orientation, and, a reverse orientation doesn't make too much sense here so won't be shown.

### Tranforming the For Loop into If-Goto Style
#### The For Loop Transformed: Original Orientation
#### The For Loop Transformed: Reverse Orientation

## Transforming the Break & Continue Structured Statements into If-Goto Style

Break and continue statements need to either exit the loop or restart the loop.&nbsp; Most loops already have a label to continue the loop; however, some loops do not have an exit label (they exit by falling though).&nbsp; Typically a break statement will require an exit label placed past the end of the loop.&nbsp; A continue label inside a for loop should execute the incrementing element (and then the condition test).

## Reversing Relational Expressions

As you've seen above, we often need to reverse the sense of a conditional test because we want to fall through on the specified condition, and take a branch on the opposite condition &#8212; this can be done by simply negating the original condition, though typically this negation costs an instruction in assembly langauge.

If the condition uses a top-level relational operator, *we can reverse the operator instead of using negation* sparing the instruction.&nbsp; Many conditional expressions often involve top-level relational operator.&nbsp; (This applies to integer comparison; floating point comparison can involve NaN, which complicate condition negation.)

In the following table, the "Reversed" column shows the negated condition.

|  Condition | Reversed |
| ------------- |:-:|
| a == b | a != b |
| a != b | a == b |
| a < b | a >= b |
| a <= b | a > b |
| a >= b | a < b |
| a > b | a <= b |

Observe that the equals sign either materializes or disappears under reversal (optimized negation) &#8212; so `!(a<b)` is `a>=b` and `!(a<=b)` is `a>b`.

So, the structured if-statement: 

    if ( a < b ) ... else ...

is logically transformed into if-goto style as:

        if ( ! (a < b) ) goto elsePart; ...

but this then optimizes to:

        if ( a >= b ) goto elsePart; ...

### Double negation

Sometimes we need to reverse a condition that already has top-level negation.&nbsp; Two top-level negtation operators can be eliminated together.&nbsp; For example, original source might be written as `! booleanFunction()` &#8212; and we might need to reverse this because the condition is part of an if-statement &#8212; in such cases, we can simply remove one negation operator instead of applying another.&nbsp; Sometimes double negation also arises in handling of short-circut boolean operators.

## Transforming Multiple Structured Statements into If-Goto Style

Transforming code as structured statements into if-goto requires labels, sometimes a lot of labels.&nbsp; Each structured statement will require one or more new labels, and in many assembly languages, these labels will have to be unique.&nbsp; 

Sometimes we'll just number labels, like L0, L1, L2, L3.&nbsp; In some assemblers, we have local labels, so that the label names/numbers can be reused in different sections of the same file, such as in different procedures.&nbsp; In other assemblers the labels will have to be unique for the entire source file.

Often we'll name and number the labels associated with different statements, i.e. `loopTop1:`, `loopExit1:`, both associated with loop 1.

Sometimes we'll name a label after what the code following does.

### Sequential Structured Statements

Two structured statements in a sequence can be transformed by simply transforming each structured statement and placing each transformation in a sequential orientation, respectively, being careful to create unique label names as needed.

### Nesting of Structured Statements

Structured statements nest nicely.&nbsp; In the if-goto style, this nesting is essentially flattened.&nbsp; This means that various sub-parts of nested structured statements will overlap &#8212; though while they will be overlap, each structured statement is still logically separate and can be reasoned about and transformed independently.

### Branches to Branches

Any time we have a branch (whether conditional or unconditional) that branches to an uncondional branch, we can optimize the code by changing the former branch to directly target the latter's label.&nbsp; Such branches to branches can arise in the rather mechanical tranformation of structured statements into the if-goto style, and, they can be eliminated as you go if you recognize them as uncesssary, or removed/optimized after final transformation, or even left as is &#8212; the code will still work; though suboptimial from a performance perspective, they are harmless.

### Consecutive Labels

Sometimes during transformation to if-goto style, we end up with two or more labels identifying the same place in the code.&nbsp; This harmless in that labels are free as they cause no machine code; however, they can be removed, if desired &#8212; though if the code changes in certain ways, duplicate labels that were removed *could* require reintroduction (I wouldn't let that stop me from removing them though, if needed I'd just put them back).

## Operand Swapping

Some hardware instruction sets are missing certain features.&nbsp; We can almost certainly substitute a longer sequence for missing features.

If a missing feature is one of the relational operators, we can substitue for a longer sequence using another relational operator in combination with negation.&nbsp; However, sometimes we can swap the operands instead.&nbsp; In `a > b` we swap to `b < a`, which tests the same condition (this is not negation) &#8212; but uses an alternate relational operator, here swapping greater-than for less--than.

In another example, `5 < b`, some hardware cannot do this in one instruction 
because they don't allow a first-operand immediate/constant, but they can do `b > 5` in one instruction, and, this is an equivalent condition.

## Short Circuit Expression Transformation: &&, || 

Beyond this, there are considerations for transcribing && and || constructs (short-cicuit `AND` and short-circut `OR`) into assembly.

This section shows how to reduce short circut boolean operators into more yet simpler if-goto statements.&nbsp; An upcoming section will show how to apply negation to these conditions, as negation is often required for if and loop transformation into if-goto style.

### Short Circut `AND`

In order to translate an expression using the `&&` operator into assembly language, we need to simplify the expression and remove the `&&` operator in favor of more if-goto statements!

        if ( a && b ) goto label;
 
Becomes:

        if ( ! a ) goto nextStatement;
        if ( b ) goto label;
    nextStatement:    
    
The `goto label;` will only fire if both `a` and `b` are true; otherwise the statement falls through to the next.&nbsp; As often happens in if-goto programming, another label is introduced for the extra flow control.

### Short Circut `OR`

Similar to the above we need to simplify such the expressions and remove the `||` operator in favor of more if-goto statements!

        if ( a || b ) goto label;

Becomes simply:

        if ( a ) goto label;
        if ( b ) goto label;
      
The `goto label;` will fire if either condition is met; otherwise the statement falls through to the next.

## Eliminating Negation

### Negating Short Circut Expressions

Conditions of the form `a && b` and `a || b` may require negation.&nbsp; Their negated forms comes from DeMorgan's laws, which distribute negation over the operands while also reversing the operator: `&&` becomes `||` and vice versa. 

| Operation | Negated | Negation Distributed |
|:-:|:-:|:-:|
| `a && b` | `! (a && b)` | `! a || ! b` |
| `a || b` | `! (a || b)` | `! a && ! b` |

One advantage to this distribution is that if either `a` or `b` involve relational operators, further short-cicut operators, or negation, then further simplifications and optimizations are enabled.

### Negation by Reversing Flow Control Branch Targets

Swapping branch targets is another way to accomplish negation.&nbsp; The pattern allows:

        if ( a ) goto someLabel;

To be negated, as such:

        if ( ! a ) goto continueAround;
        goto someLabel;
    continueAround:
    
*Let's also note that in the above, `a` could represent something that already has negation, so let's show that broken out:*

        if ( ! b ) goto someLabel;

*To be negated, we simply remove the existing negation (total of zero negations) instead of introducing another negation (total of 2 negations):*

        if ( b ) goto continueAround;
        goto someLabel;
    continueAround:
    
## Applying Patterns in a Series of Mathematical Transformations

To see how this comes together let's look at conjunction, `&&`, in the condition of a structured loop, here a loop searching an array of 100 for the first zero slot:

    while ( i < 100 && a[i] != 0 ) { i++; }

Let's translate this loop statement to if-goto style using original orientation:

    loop:
        if ( ! (i < 100 && a[i] != 0) ) goto exitTheLoop;
        // body
        i++;
      c goto loop;
    exitTheLoop:

### Approach 1: Eliminate Negation Using De Morgan 

We'll now need to simplifiy the conditional expression:

    ! (i < 100 && a[i] != 0)

which &#8212; using deMorgan to distribute the negation &#8212; becomes:

      ! (i < 100) || ! (a[i] != 0)
      
which &#8212; using negation of relational operators (twice) &#8212; becomes:

        i >= 100 || a[i] == 0
       
So, we'll put that series of simplifcations back into context, as:

        if ( i >= 100 || a[i] == 0 ) goto exitTheLoop;

and by the above short-circut evaluation of `||` in if-goto:

        if ( i >= 100 ) goto exitTheLoop;
        if ( a[i] == 0 ) goto exitTheLoop;

This pair of simplified if-goto statements (short-circut boolean operators removed) accomplishes the same as the above (`if ( ! (i < 100 && a[i] != 0) ) goto exitTheLoop;`).

### Approach 2: Eliminate Negation Using Branch Target Swapping

Using this pattern, let's reverse the same expression:

        if ( ! (i < 100 && a[i] != 0) ) goto exitTheLoop;

We can remove the negation by changing the branch target (which requires introducing a label, `continueThis:`, and an unconditional branch):

        if ( i < 100 && a[i] != 0 ) goto continueThis;
        goto exitTheLoop;
    continueThis:

Next, we'll apply short-circut evaluation of `&&` in if-goto (which also requires introducing another label, `nextPart:`).

        if ( ! (i < 100) ) goto nextPart;
        if ( a[i] != 0 ) goto continueThis;
    nextPart:
        goto exitTheLoop;
    continueThis:

Now we'll remove negation by swapping the relational operator around:

        if ( i >= 100) ) goto nextPart;
        if ( a[i] != 0 ) goto continueThis;
    nextPart:
        goto exitTheLoop;
    continueThis:

And we'll apply branch to branch optimization (a branch to an unconditional branch can be modified to go directly to the final target), changing the target of the first goto from `nextPart` to `exitTheLoop`.&nbsp; (This also allows removing the `nextPart` label as it is no longer used.)

        if ( i >= 100) ) goto exitTheLoop;
        if ( a[i] != 0 ) goto continueThis;
        goto exitTheLoop;
    continueThis:

Now, the second conditional branch (if-goto) can be negated in order to swap branch targets.&nbsp; Whereas initially in this section, we introduce swapped branch targets in order to eliminate negation, this time, we're introducing negation in order to swap branch targets!&nbsp; (This also allows removing the label `continueThis`.)

        if ( i >= 100) ) goto exitTheLoop;
        if ( ! (a[i] != 0) ) goto exitTheLoop;

And lastly, we can remove the negation by reversing the relational operator:

        if ( i >= 100) ) goto exitTheLoop;
        if ( a[i] == 0 ) goto exitTheLoop;

So, this represents another way to reach the same code sequence.

---

This article is Copyright (c) 2018-2019, Erik L Eidt.
