
When I was in high school, parsing expressions in text facinated me.&nbsp;
 I determined an algorithm for this, and I wrote a small interpreter.&nbsp; 
 It is no wonder that I wound up in compiler technology, eventually finding similar facination in code generation and optimization.
 
I developed this algorithm without formal training in computer science &#8212; or science alone for that matter!&nbsp; 
 Since then, of course, I have discoved that my "invention" was not wholly unique, as these things have been topics of interest for ages.
 
However, I believe that my approach still has merits worth discussing, though I discuss now in the context of what I later realized are
previously existing approaches.

The classic Shunting Yard algorithm is incomplete for expressions with certain operators found in the C style of languages.&nbsp; 
 It does not allow for parsing subtraction together with negation and negative numbers.&nbsp;
 It does not provide for function calls and array indexing.&nbsp;
 Further, the classic Shunting Yard algorithm does not properly detect a number of syntax errors, such as repeated operands.
 
In some sense, dealing with these requires two shunting yard algorithms, stitched together.&nbsp;
 My approach to parsing all these operators is a two-state, two-stack model.&nbsp;
 The two states are the Unary State and the Binary State, and, the two stacks are Operator Stack and the Operand Stack.

Using these four elements, we can precisely identify the difference between parenthesis for grouping vs. for function calls,
 and minus sign for negation vs. subtraction.&nbsp; 
 This is possible because the grouping parenthesis occurs in the unary state, 
  whereas the function call parenthesis occurs in the binary state.&nbsp;
 Similarly, subtraction occurs in the binary state, but negation occurs in the unary state.
 
Also noteworthy, this approach to parsing complex expressions involves no recursion; 
 each token is directly dealt with and the algorithm moves on to process the next token.&nbsp; 
 By contrast, recursive algorithms, and even those based on flattened recursion, generally look at each token 
  numerous times as they analyze its applicability to each level of precedence.&nbsp;
 Thus, I have found this approch to have considerable performance especially in languages that have 
  a large number of levels of precedence, like C.
  
The algorithm starts in the Unary State, with an empty Operator Stack and an empty Operand Stack.&nbsp; 
 When the algorithm completes the result of the expression parse is the (only) operand on the Operand Stack.

The gist of the algorithm is that in each state certain things are expected (and differentiated).&nbsp; 
 Upon receipt of tokens, they are handled as per the state, and then the state either remains or switches to the other,
  depending on the operator.

For example, receiving an open parenthesis in the (initial and) unary state, means we have an ordinary grouping
 parenthesis &#8212; and we stack it and remain in the unary state, so as to accept more open parenthesis, or a unary operator,
 like minus sign, or an identifier or constant.

Receiving parenthesis in the Binary State means the beginning of a function call rather than grouping.&nbsp;
 We stack the function invocation and transition to the Unary State.
 


