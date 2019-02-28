
When I was in high school, parsing expressions in text facinated me.&nbsp;
 I determined an algorithm for this, and I wrote a small interpreter.&nbsp; 
 It is no wonder that I wound up in compiler technology, eventually finding similar facination in code generation and optimization.
 
I developed this algorithm without formal training in computer science &#8212; or science alone for that matter!&nbsp; 
 Since then, of course, I have discoved that my "invention" was not wholly unique, as these things have been topics of interest for ages.
 
However, I believe that my approach still has merits worth discussing, though I discuss now in the context of what I later realized are
previously existing approaches.

The classic Shuning Yard algorithm is incomplete for expressions with additional operators.&nbsp; 
 It does not allow for parsing subtraction together negation and negative numbers.&nbsp;
 It does not provide for function calls and array indexing.
 
In some sense, these additional operators require two shunting yard algorithms, stitched together.&nbsp;
 My approach to parsing all these operators is a two-state, two-stack model.&nbsp;
 The two states are unary and binary, and, the two stacks are operator and operand.

Using these four elements, we can precisely identify the difference between parenthesis for grouping vs. for function calls,
and minus sign for negation vs. subtraction.&nbsp; 
 This is possible because the grouping parenthesis occurs in the unary state, 
 whereas the function call parenthesis occurs in the binary state.&nbsp;
 Similarly, subtraction occurs in the binary state, but negation occurs in the unary state.
 
Also noteworthy, this approach to parsing complex expressions invovles no recursion; 
 each token is somehow dealt with and the algorithm moves on directly.&nbsp; 
 By contrast, recursive algorithms and even those based on flattened recursion generally look at each token 
  numerous times as they analyze the applicability of each level of precedence.&nbsp;
 Thus, I have found this approch to have considerable performance especially in languages that have 
  a large number of levels of precedence, like C.
  



