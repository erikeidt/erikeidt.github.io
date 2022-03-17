# The Double-E Infix Expression Parsing Method

## Introduction 

This article describes the Double-E Infix Expression Parsing Method.&nbsp; 
Infix expressions are the kind found in most programming languages, like C.

The Double-E method uses two states and two stacks.

The states are simple: unary and binary.&nbsp; 
To oversimplify slightly, the unary state says we're looking for a unary operator or an operand, and the binary state says we're looking for a binary operator or the end of an expression.

One of the two stacks is for operators and the other for operands.&nbsp; 
These stacks are intermediate storage used to hold operators and operands before their proper precedence and binding is known.&nbsp; 

An element of the operator stack is a simple enum describing an operator.&nbsp; 
These differentiate between unary negation and subtraction, for example, so is a stack of true operators in the expression language, rather than just tokens.

An element of the operand stack is a tree node, which represents an expression.&nbsp; 
Tree nodes can represent constants, identifiers, addition of two operands, etc.&nbsp; 
Such a tree node can be called an Abstract Syntax Tree, AST, so the operand stack is a stack of ASTs.

When proper binding of operators to operands is discovered, those operators and their operands are removed from these stacks and replaced.&nbsp; 
The replacement is one tree representing the operators bound to their operands, and this replacement is is pushed onto the operand stack.

When the parse successfully completes, then an AST representing the entire expression is left on the operand stack.

This method is:

* Simple  
	* The two states are easily reflected in code; there is no need for complicated state machine
	  * depending on next input, simply stay in current state or switch to the other
	  * current state tells us whether operator is unary or binary
	* Two simple stacks
	  * the stacks are used to manage precedence and bind partial expressions to operators

* Complete
	* Handles unary negation, indirection, subtraction, multiplication, prefix, postfix, ternary operators.
	* Handles all operators of C, such as grouping parenthesis, function invocation, array referencing.
	* Differentiates between f(a,b) and f((a,b)).
	* Can be expanded to handle lambdas, casts, other.

* Efficient
	* Simple loops for the states, no recursion, no backtracking.
	* Each *token* is examined only once, and immediately converted to an operator or operand
	  * and placed on one of the two stacks.
	* Can work directly on characters; does not need a formal lexer.

* Composable 
	* Composes with recursive descent statement parser

* Implementation
	* [Draconum Double-E Expression Parser](https://github.com/erikeidt/Draconum/tree/master/src/3.%20Expression%20Parser)
	  * [ExpressionParser.cs](https://github.com/erikeidt/Draconum/blob/master/src/3.%20Expression%20Parser/Expression%20Parser%20Library/ExpressionParser.cs)

## Notes

When starting the parse of an infix expression, the initial state is the unary state, and the stacks are both empty.

In the unary state, we're expecting either a unary operator or an operand or grouping parenthesis (or others).&nbsp; If we:

* see an operator token (e.g. -, \*), then we know it is a unary operator (e.g. unary negation, indirection).&nbsp; Push the identified unary operator onto the operator stack.&nbsp; Stay in unary state.

* see an operand (e.g. identifier, constant), so create AST for the operand and push it onto the operand stack.&nbsp; Switch to binary state.

* see an open paranthesis, then push operator for grouping paren onto the operator stack.&nbsp; Stay in unary state.

In the binary state, we are expecting binary operators, or close paranthesis (or open paren).&nbsp; If we:

* see an operator token (e.g. - or \*) in the binary state, and we know we have a binary operator (e.g. subtraction or multiplication).

  The idea now is to reduce the operator stack as much as possible, then push the identified operator.&nbsp;
Reduction is a matter of composing an AST for an expression on the stacks as long as the precedence of the 
operator on top of the operator stack is greater (greater or equal for right associative) than the precedence of the newly identified operator.&nbsp;
The reduction builds ASTs by popping operators and as many operands as the operators take, 
off the stacks, and pushing the newly constructed partial tree back onto the operand stack.&nbsp; Switch back to unary state.

* see a close paranthesis, then reduce until matching open parenthesis.

  Note: the matching open paren may be a function invocation operator, if so build function invocation tree node.&nbsp; 
If not, discard grouping paren.&nbsp; Stay in binary state.

* see an open parenthesis, that is a function invocation.

  Reduce.&nbsp; Push function invocation operator onto the operator stack.&nbsp; Switch to unary state.

That's the gist of it.  

# Example

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
operand stack: identififer a

input: plus sign token (+)
3: binary addition operator: reduce until precedence of addition is higher than that on the operator stack
   that means constructing a tree for unary negation of identifier a,
   popping unary negation off the operator stack, popping identifier a off the operand stack
   and pushing the AST for unary negation of identifier a onto the operand stack
   push operator binary addition onto the operator stack

state: unary
operator stack: binary addition
operand stack: AST for -a

input: identifier b
4: push operand identifier b onto the operand stack

state: binary
operator stack: binary addition
operand stack: AST for -a, AST for b

input: asterisk token (*)
5: reduce (nothing to do: binary multiplication has higher precedence than binary addition)
   push binary multiplication onto operator stack

state: unary
operator stack: binary addition, binary multiplication
operand stack: AST for -a, AST for b

input: identifier c
6: push operand c

state: binary
operator stack: binary addition, binary multiplication
operand stack: AST for -a, AST for b, AST for c

input: semi colon token (;)
7: end of parse (semi colon unrecognized, so not consumed).
   (it is legal to end parse when in binary state)
   Reduce until single operand.&nbsp; Parse error if cannot reduce to 1, if unmatch paren, etc..
   
   Reducing will first remove the binary multiplication from the stacks, resulting in:

state: binary
operator stack: binary addition
operand stack: AST for -a, AST for b * c

   Next the addition is replaced by an AST representing the addition.
   
state: success
operator stack: empty
operand stack: AST for -a + b * c

      +
    /   \
   -     *  
   |    / \
   a   b   c

```
