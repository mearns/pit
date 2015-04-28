A scope is an object which has a dictionary and a pointer to another scope,
it's _parent_ scope. A namespace is a concept that encapsulates all recognized
symbols in a particular execution context. It is represented by a scope
object, but encapsulates all of the linked scopes.

Through this linking, the scope forms a linked list, which implements a stack,
with the initial scope (at the start of the list) representing the top of the
stack.

All executed code has an associated namespace (represented by a scope obejct).
To resolve the value (or existence) of a symbol in that code, the first scope
in the namespace (the top of the stack) is searched first for the symbol. If
it doesn't exist there, we recursively search through the parent scope if
there is any. If we get to the end of the linked list of scopes without
finding it, then it doesn't exist.

A similar path is taking for assigning a value to a variable, and if it
doesn't exist in any of the scopes, it is created in the first scope (the top
of the stack), which represents the local scope.

All executable blocks are first class objects. When they are created, the
current namespace is attached to that object, and anytime that block is
executed, it is executed in that namespace.

Curly brace pairs around statements define `blocks`, which are first class
executable objects. They are not executed when they are defined, they act as
values. Blocks form the basis for all executable objects.

As described, blocks execute in the namespace in which they are defined. This
is done by associating the current scope at the top of the stack with the
block object. Note that the block is associated with a scope object, so as
additional scopes are pushed onto the namespace in the containing code, they
are _not_ included in the block's namespace. The added scopes include that
namespace (and therefore code that executes in those scopes may modify its
contents), but the scope attached to the block is further up the stack and has
no way to reach the added scopes.

Calling code can also add additional variables which are added to an other
wise empty scope pushed onto the stack. Lastly, an addition empty scope is
pushed on just before the block is executed, for local variables inside the
block. Both of these added scopes are popped when the block ends.

You can define a function simply by assigning a block to a variable or
constant, for instance. In fact, any other way to define a function is simply
a wrapper or syntactic sugar around this construct. One such wrapper allows
you to specify the args that are expected. This wrapper actually creates an
additional block which checks the arguments before executing the wrapped block
(arguments are just syntactic sugar for pushing caller defined variables onto
the execution namespace).

Similarly, methods are just wrappers around blocks that automatically push the
associated object as the first argument, so they end up as implicit first
arguments.

`while` is not actually a keyword, but a built-in function which takes two
blocks as arguments. The first is the test, the second is the loop nody.
Similarly with `for` loops and conditionals. The loop constructs invoked with
fewer arguments than expected create _partials_. So for instance, `while`
invoked with a single block returns a new function which accepts a single
argument and executes it over and over as long as the origin block evaluates
as true.

When executed, the result of the last statement in the block becomes the
resulting value. It is said that the block _evaluates_ as the result.

This is _separate_ from a `return` statement. The `return`, `break`, and
`continue` keywords actually cause exceptions to be raised, which bubble up
through the stack until they are handled. Most function wrappers handle the
return except by causing the function to evaluate as the returned value
(which is contained by the ReturnException). Loop constructs handle
BreakExceptions and ContinueExceptions appropriately, so these correctly end
execution of the block and cause the loop to work as expected. They do not
generally handle ReturnExceptions, so if you `return` inside a loop, for
instance, it will most likely bubble all the way up to the enclosing function
and return from it, as expected.

A `frame` is an execution frame and has an associated namespace, represented
by a scope object.

