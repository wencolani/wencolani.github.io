Yue Zhang , Joakim Nivre     ACL2011
#Motivation
improve the accuracy of traditional-based dependency parsers by considering even richer feature sets than those employed in previous systems. 
# the Transition-based Parsing Algorithm

typical transition-based parsing process: the input words are put into a quenue and partially built structures are organized by a stack. A set of shiftreduce actions are defined, which consume words from the quenue and build the output parse.

## arc-eager system:
*  **shift**: removes the front of the queue and pushes it into the top of the stack;
*  **Reduce**: pops the top item off the stack;
*  **LeftArc**: pops the top item off the stack and adds it as a modifier to the front of the queue;
*  **LeftArc**: removes the front of the queue, pushes it into the stack and adds it as a modifier to the top of the stack.
