# The Double-E Infix Expression Parsing Method

## Introduction 

This article describes the Double-E Infix Expression Parsing Method.

This is a method for parsing infix expressions such as we find in most languages like C, having many levels of precedence, and a rich set of operators with static operator precedence.&nbsp; It is a bottoms-up approach to parsing similar to general purpose shift-reduce parsing, though dedicated to and simplified for infix expressions.

It does not use state tables, recursion, or backtracking.&nbsp; Parse state tables are replaced by two states, which are reflected in two code sections rather than as data tables.&nbsp; Instead of recursion, there are two simple stacks.&nbsp; It looks at each input item once and handles it directly &#8212; it does not re-examine input; it does not backtrack.&nbsp; It is industrial strength accepting what it should and rejecting what it shouldn't, yet as simple as the Shunting Yard algorithm (yet without the well-known limitations: operator support and error detection).

It is also extensible, for example, with a simple change, what are errors in my implementation, can detect missing binary operator (e.g. juxtaposition as in `a b`), similarly missing operand `f(,x)`.

The Double-E method uses two states and two stacks.

The states are simple: Unary and Binary.&nbsp; 
To oversimplify, the Unary State says we're looking for a unary operator or an operand, and the Binary State says we're looking for a binary operator or the end of an expression.

One of the two stacks is for Operators and the other for Operands.&nbsp; 
These stacks are intermediate storage used to hold operators and operands before their proper precedence and binding is known.

An element of the Operator Stack is a simple enum describing an operator.&nbsp; The enum differentiates between unary negation and substraction, for example, so, the operator stack is a stack of true operators in the expression language, rather than input tokens.

An element of the Operand Stack is a tree node, which represents an expression.&nbsp; 
Tree nodes can represent constants, identifiers, addition of two operands, etc.&nbsp; 
Such a tree node can be called an Abstract Syntax Tree, AST, so the operand stack is a stack of ASTs.

When proper binding of operators to operands is discovered, those operators and their operands are removed from these stacks and replaced.&nbsp; 
Each replacement is a single tree representing the operators bound to their operands, and this replacement is pushed onto the operand stack.

When the parse successfully completes, then one AST representing the entire expression is left on the operand stack.

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

I developed this algorithm in 1978.&nbsp; The closest thing I've see to this is the Shunting Yard algorithm, but that is lacking significant capability.&nbsp; We might describe the Double-E algorithm as gluing two Shunting Yard algorithms together (one for each of the two states, while also capturing more parse state in the states & stacks).&nbsp; Shift-reduce parsing is more general purpose approach, though state tables introduce complexity that isn't needed for infix expression parsing, which arguably makes it harder to understand.


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

# Example

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
operand stack: identifier a

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
