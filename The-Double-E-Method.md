# The Double-E Infix Expression Parsing Method

## Introduction 

This article describes the Double-E Infix Expression Parsing Method.

Key to understanding infix expressions is the observation that there are two frequently alternating states.&nbsp; I call these states Unary and Binary; we could also describe them as Operand and (binary) Operator.&nbsp; At the beginning of an expression we expect and operand, and once we have an operand we expect an operator; after an operator we again expect an operand.&nbsp; This simple alternation of operand & operator (aka Unary & Binary states) allows complete decomposition of infix expressions making them easy to parse.

If you are interested in parsing expressions, such as those found in C, namely having many levels of (statically known) precedence among operators, having paranthesis for grouping and for function calls, having unary operators and binary operators, techniques such as recursive decent and Pratt parsing offer a top-down approach.&nbsp; This article is about a rather simple alternative using a bottoms up approach.

This bottoms-up approach to parsing is similar to the more general purpose shift-reduce parsing, though dedicated to and simplified for infix expressions.

This technique does not use state tables, recursion, or backtracking.&nbsp; Parse state tables are replaced by two simple states &#8212; and these two states are embodied as two separate code sections.&nbsp; Instead of recursion, there are two simple stacks.

This also approach mixes nicely with any statement parser, such as one using recurive decent.

This expression parser looks at each input item once and handles it directly &#8212; it does not re-examine input; it does not backtrack.&nbsp; It is industrial strength accepting what it should and rejecting what it shouldn't, yet as readable as the Shunting Yard algorithm (though without its well-known limitations of operator support and error detection).

The algorithm is also extensible, for example, with minor change, what are errors issued in my implementation, can detect and handle missing binary operator (e.g. juxtaposition as in `a b`), and similarly missing operand `f(,x)`.

---

The Double-E method uses two states and two stacks.

The states are: Unary and Binary. 

To oversimplify, the Unary State says we're looking for a unary operator or an operand, and the Binary State says we're looking for a binary operator or the end of an expression.

Of the two stacks, one is for Operators and the other for Operands.&nbsp; 
These stacks are intermediate storage used to hold operators and operands before their proper bindings are known in context of the larger expression.

An element of the Operator Stack is a simple enum describing an operator.&nbsp; 
In other words, the operator stack is a stack of operators in the expression language, 
for example, differentiating between unary negation and binary subtraction.&nbsp; 
(Let's note that input tokens or characters, do not so differentiate.)&nbsp; 
The operator stack holds operators that have been seen and are not yet bound to their operands.

An element of the Operand Stack is a tree node, which represents an expression, or a part of an expression that has been recognized.&nbsp; 
Tree nodes can represent constants, identifiers, addition of two operands, etc.&nbsp; 
Such a tree node can be called an Abstract Syntax Tree, AST, so the operand stack is a stack of ASTs.&nbsp; 
The operand stack holds expressions that are in various states of recognition, ranging from one simple node to complete expression tree.

When proper binding of operators to operands is discovered/recognized, 
those operators and their operands are removed from these stacks and a tree composed from them is created as a replacement.&nbsp; 
A replacement is a single tree representing the operators bound to their operands, and this replacement is pushed onto the operand stack.
This process of recognizing a binding (of operands to operators), and replacing those stacked parts with a single tree is called reduction or a reduction.&nbsp;
(Though it involves a composition of a new tree operand, it is a reduction of the intermediate state and associated complexity based on some recognition.)

When the parse successfully completes, then one AST representing the entire expression is left on the operand stack.&nbsp;

---

This method is:

* Simple  
	* The two states are easily reflected in code alone; there is no need for complicated state machine
	  * depending on next input, simply stay in current state or switch to the other
	  * current state tells us whether operator is unary or binary
	* Two simple stacks
	  * the stacks are used to manage precedence and bind partial expressions to operators
	* No additional classes/objects are required for parsing state, beyond those for the output AST data structure and the two stacks.
	  * Most of the parsing happens within a single function.

* Complete
	* Handles unary negation, indirection, subtraction, multiplication, prefix, postfix, ternary operators.
	* Handles all operators of C, such as grouping parenthesis, function invocation, array referencing.
	* Differentiates between f(a,b) and f((a,b)).
	* Can be expanded to handle lambdas, casts, juxtaposition, other.

* Efficient
	* Simple loops for the states, no recursion, no backtracking.
	* Each *token* is examined only once, and immediately converted to an operator or operand
	  * and placed on one of the two stacks.
	* Can work directly on characters; does not need a formal lexer.
	* Beyond the (reusable) stacks, the only objects created are the AST's for parse output.

* Composable 
	* Composes with recursive descent statement parser

* Implementation
	* [Draconum Double-E Expression Parser](https://github.com/erikeidt/Draconum/tree/master/src/3.%20Expression%20Parser)
	  * [ExpressionParser.cs](https://github.com/erikeidt/Draconum/blob/master/src/3.%20Expression%20Parser/Expression%20Parser%20Library/ExpressionParser.cs)

I developed this algorithm in 1978.&nbsp; The closest thing I've see to this is the Shunting Yard algorithm, but that is lacking significant capability.&nbsp; We might liken the Double-E algorithm as gluing two Shunting Yard algorithms together (one for each of the two states, while also capturing more parse state in the states & stacks).&nbsp; Shift-reduce parsing is more general purpose approach, though the state tables introduce complexity that isn't needed for infix expression parsing.

The Double-E approach is simple & accurate; it uses code instead of a grammar.&nbsp; 
This, along with no recursion, no backtracking, helps keeps things simple.&nbsp; 
To accept a different expression language, we edit the code.&nbsp; 
It produces AST's, but not Parse Trees aka Concrete Syntax Trees.&nbsp; 
(One could add tree nodes for parenthesis as though are elided from generated ASTs, for example).

---

## Notes

When starting the parse of an infix expression, the initial state is the unary state, and the stacks are both empty.

In the unary state, we're expecting either a unary operator or an operand or grouping parenthesis (or others).&nbsp; If we see:

* an operator token (e.g. -, \*), then we know it is a unary operator (e.g. unary negation, indirection).&nbsp; Push the identified unary operator onto the operator stack.&nbsp; Stay in unary state.

* an operand (e.g. identifier, constant), so create AST for the operand and push it onto the operand stack.&nbsp; Switch to binary state.

* an open parenthesis, then push operator for grouping paren onto the operator stack.&nbsp; Stay in unary state.

In the binary state, we are expecting binary operators, or close parenthesis (or open paren).&nbsp; If we see:

* an operator token (e.g. - or \*) in the binary state, and we know we have a binary operator (e.g. subtraction or multiplication).

  The idea now is to reduce the operator stack as much as possible, then push the identified operator.&nbsp;
Reduction is a matter of composing an AST for an expression on the stacks as long as the precedence of the 
operator on top of the operator stack is greater than the precedence of the newly identified operator (greater or equal for right associative).&nbsp;
The reduction builds ASTs by popping operators and as many operands as the operators take, 
off the stacks, and pushing the newly constructed partial tree back onto the operand stack.&nbsp; Switch back to unary state.

* a close parenthesis, then reduce until matching open parenthesis.

  Note: the matching open paren may be a function invocation operator, if so build function invocation tree node.&nbsp; 
If not, discard grouping paren.&nbsp; Stay in binary state.

* an open parenthesis, that is a function invocation.

  Reduce.&nbsp; Push function invocation operator onto the operator stack.&nbsp; Switch to unary state.

That's the gist of it.  

A parse completes by running into a token that isn't part of the expression language.&nbsp; 
For example, in C, a ';', the semi colon terminates an expression statement.&nbsp; 
Since the semi colon is not an operator in the expression langauge, the parse will stop rather than consume that character/token.&nbsp; 
It is up to the caller (e.g. the statement parser) to then consume the semi colon.

As the parse completes, it reduces everything that is left on the stacks.&nbsp; 
The operator stack should become (or be) empty.&nbsp; 
If the operator stack is not empty, there is an error in the expression being parsed.&nbsp; 

For example, "a+" is an error of missing operand: the reduction attempts to reduce the + operator but there are not enough operands on the operand stack, so error.
Further, "(a+b" is an error of missing parenthesis: normally the '(' open paren is matched by a closing paren ')' and thus removed from the operator stack.&nbsp; 
So, if we run into one trying to reduce, it was unmatched in the input expression.&nbsp; 
While "a+b)" is an unmatched parenthesis, as it's mate cannot be found on the operator stack.

Note that the parser can be told to support a parse terminating character, which is done for example, when parsing a C-style if statemtent that has the ( ) built in (requried).&nbsp;
The parser will simply stop on an unmatched ')' instead of issuing an error for that.&nbsp; 
A statement parser will have consumed required the initial '(' (e.g. of "if ( ) { } else { }"), asked the expression parser to stop on ')', and is left to also consume the final ')' of the if statement condition expression.

In my incarnation, the expression parser issues meaningful error messages that fail the parse; 
however, with modification, it could return the error message along with a failure indication to allow the caller to proceed in the face of expression errors.

---

# Example 1

Let's parse: -a + b*c;

```
 u   u   b   u   b   u   b		States
  \ / \ / \ / \ / \ / \ / \     	\ for current state ready for input, / for transitions to next state
   -   a   +   b   *   c   ;		Sample input
   1   2   3   4   5   6   7		Action (comment #), given current state & input

initial state: unary
operator stack: empty
operand stack: empty

input: minus sign token (-)
1: push unary negation enum onto operator stack; stay in unary state

state: unary
operator stack: unary negation
operand stack: empty

input: identifier a
2: create AST for identifier a, and push that onto operand stack; switch to binary state

state: binary
operator stack: unary negation
operand stack: (AST for) identifier a

input: plus sign token (+)
3: binary addition operator: reduce until precedence of addition is higher than that on the operator stack
   that means constructing a tree for unary negation of identifier a,
   popping unary negation off the operator stack, popping identifier a off the operand stack
   and pushing the AST for unary negation of identifier a onto the operand stack
   push operator binary addition onto the operator stack
   switch to unary state

state: unary
operator stack: binary addition
operand stack: AST for -a

input: identifier b
4: push operand identifier b onto the operand stack; switch to binary state

state: binary
operator stack: binary addition
operand stack: AST for -a, AST for b

input: asterisk token (*)
5: reduce (nothing to do: binary multiplication has higher precedence than binary addition)
   push binary multiplication onto operator stack; switch to unary state

state: unary
operator stack: binary addition, binary multiplication
operand stack: AST for -a, AST for b

input: identifier c
6: push operand c; switch to binary state

state: binary
operator stack: binary addition, binary multiplication
operand stack: AST for -a, AST for b, AST for c

input: semi colon token (;)
7: end of parse (semi colon unrecognized, so not consumed).
   (it is legal to end parse when in binary state)
   Reduce until single operand. Parse error if cannot reduce to 1, if unmatch paren, etc..
   
   Reducing will first remove the binary multiplication from the stacks, resulting in:

operator stack: binary addition
operand stack: AST for -a, AST for b * c

   Next the addition is replaced by an AST representing the addition.
   
operator stack: empty
operand stack: AST for -a + b * c

    exit the parse, yielding the one operand remaining on the operand stack:

      +
    /   \
   -     *  
   |    / \
   a   b   c

```

---

# Example 2

Let's parse (a+b)*f(c,d)

```
initial state: unary
operator stack: empty
operand stack: empty

input: open paren (
1: push grouping paren enum onto operator stack; stay in unary state

state: unary
operator stack: grouping paren (
operand stack: empty

input: identifier a
2: create AST for identifier a, and push that onto the operand stack; switch to binary state

state: binary
operator stack: grouping paren (
operand stack: (AST for) identifier a

input: plus sign +
3: binary addition operator: reduce until precedence of addition is higher than that on the operator stack
   note that the search of the operator stack (backward, i.e. from top of stack) for possible reductions is halted by the presence of the (
   this means that any operators further toward the bottom of the stack are precluded from reduction at this point, even if they are higher precedence than +
   push operator binary addition onto the operator stack
   switch to unary state

state: unary
operator stack: grouping paren (, binary addition
operand stack: AST for a

input: identifier b
4: push operand identifier b onto the operand stack as AST; switch to binary state

state: binary
operator stack: grouping paren (, binary addition
operand stack: AST for a, AST for b

input: close paren )
5: reduce operators until matching )
   here that means popping the + and two operands off the stacks, while creating an AST for a+b
   otherwise, do nothing more with the ), and also pop the matching ( off the operator stack
   push the AST for a+b onto the operand stack
   stay in binary state

state: binary
operator stack: empty
operand stack: AST for a+b

input: asterik token *
6: reduce until higher precedence than * (nothing to do as the operator stack is empty)
   push binary multiplication onto the operator stack
   switch to unary state

state: unary
operator stack: binary multiplication *
operand stack: AST for a+b

input: identifier f
7: push AST for f onto the operand stack
   switch to binary state

state: binary
operator stack: binary multiplication *
operand stack: AST for a+b, AST for identifier f

input: open paren (
8: seeing ( in the binary state means this is function application
   reduce operators (on the operator stack) that are higher precedence than function application (here, there are none)
   push function call ( onto the operator stack
   switch to unary state

state: unary
operator stack: binary multiplication *, function call (
operand stack: AST for a+b, AST for identifier f

input: identifer c
9: push AST for identifier c onto the operand stack
   switch to binary state

state: binary
operator stack: binary multiplication *, function call (
operand stack: AST for a+b, AST for identifier f, AST for identifer c

input: comma token ,
10: reduce all operators higher precedence than comma (none)
    push comma operator onto the operator stack
    switch to unary state

state: unary
operator stack: binary multiplication *, function call (, comma operator ','
operand stack: AST for a+b, AST for identifier f, AST for identifer c

input: identifier e
11: push AST for e onto the operand stack
    switch to binary state

state: binary
operator stack: binary multiplication *, function call (, comma operator ','
operand stack: AST for a+b, AST for identifier f, AST for identifer c, AST for indentifier d

input: close paren )
12: reduce until matching (
    we construct a comma operator tree for c,d
    the matching ( is a function call '(', which means that
    we convert all top-level ',' operators into argument separator, which are later handled differently than regular ',' operator
        this doesn't necessarily have to be done here but I choose to differentiate in generated AST between the two ',' meanings
    and we convert the function call '(' operator into a function call operator (pop & push)
        (unlike with a grouping paren, which would be removed)
    stay in binary state

state: binary
operator stack: binary multiplication *, function call operator
operand stack: AST for a+b, AST for identifier f, AST for parameter expression c,d

input: <eof> (end of file/input)    
13: end of parse (eof not recognized so not consumed)
    reduce all operators, leaving only empty operator stack and one AST on the operand stack
    this means making an AST for f(a,b)
    then making an AST for the multiplication: a+b * f(c,d)

      *
   /     \ 
  +     fn call operator (left child is function to call, right child is argument list)
a   b     / \
         f   ,
            / \
           c   d

```
