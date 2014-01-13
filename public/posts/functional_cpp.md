Recently I'm playing with [libcppa](https://github.com/Neverlord/libcppa), by
implementing the [Raft consensus protocol](https://ramcloud.stanford.edu/wiki/download/attachments/11370504/raft.pdf).

When I am writing tests, I become more and more certain that classes is not the
right pieces to make up a program. A class, or its encapsulation by using private
members and methods can make some tests hard or impossible. If you lift the
members to public, then the class is not necessary any more.

<!-- more -->

Raft is a consensus protocol, meaning it can make different nodes across the
network to agree with things. Any participant has to be in 1 of 3 modes: the
follower, the candidate, and the leader. Only the leader answers to requests
from clients. All the others are followers, and they only observe and acknowledge
to logs from the leader generated after accepting write requests. They don't talk
to the clients, not even read requests. Which means, there is no way to tell if
the internal states of the follower is correct, as there is no such interface.

To test the follower behavior, I either have to peek at its state after each
transaction, or execute another transaction to see if the second one works and
guess that the state must be correct. Well, why bother, so I take the first
approach. But how can I access the state if it's a private member of the
```raft``` class? Then there's another choice to make:

1. make the test suite a friend of the class;
1. make the state field public, or make a public accessor;
1. make the state field public when it's testing;
1. forget it, don't define a ```raft``` class in the first place.

Obviously, since I am writing this post, I definately took the last option.
Friends are uglym and I don't like tests forcing things into the tested code.
Also, having different code for tests and production immmediately sounds like a
terrible idea, as the production code is actually not tested. Though, wrong
accessibility of fields should be catchable by the compiler, so it's not such a
huge problem by itself, but it still smells. Making a public accessor is a bad
idea as nothing besides the tests will require it. Making the state field public
directly breaks encapsulation, and renders the class wrapper unnecessary.

When we have closures, why do we need classes any more? C doesn't have classes,
but it's still possible to achieve polymorphism in C, via function pointers. And
closures are the upgraded version of function pointers, with states to get rid of
the horrible ```void* arg``` every time we define a callback.

I've read somewhere another day, that Haskell, with closures, doens't need
classes in the OO sense. The argument is that, (in principle, as I cannot find
the link now), classes is basically a way to bind states to functions; since
closure is another way to do it, and more reflexible, there is no need to define
a class every time we need a small difference in behavior. We can just define a
lambda function, capture whatever state it needs, and pass it on. It cannot be
more insightful. Think of the anonymous classes in Java. And think of factory
methods. Class names are rarely used, only to instantiate. Later when being used,
the exact class names don't appear in user code, or polymophism doesn't work.

As the 3 modes transit to each other within the state machine, the implementation
of each mode should actually be private. To make them testable, I broke the
```raft``` class into 3, 1 for each mode. Also, they're not classes. They are
each a function returning the behavior for the libcppa actors.

Breaking down the class actually helps make the code more mock-testable. If I
went with the class, or classes, much of the infrastructure would definately go
into the class/classes. Like log reading/writing, or random timeout generation.
When testing, to have to read/write logs from/to the disk is too inconvenient,
even if you don't have to initialize the log files beforehand, and remove them
afterwards. It should be made ```virtual``` to be overridden in the derived mock
class. However, it's more propable that I will go with the real log processing
implementation and test them together. However, the fact that log processing
itself should have its own test set. Indirect tests may not give the most
thourough coverage. Also, it's highly impossible I would make random timeout
generation ```virtual```, but being able to change the timeout distribution is
still useful, say to test conflict cases when multiple candidates share similar
election deadlines.

Closures instead of classes make switching strategies/policies easier. With a
```candidate``` class, if I want to ```become``` it when the ```follower``` times
out, the ```follower``` behavior must templated to be able to instantiate
different ```candidate``` classes on demand. With closures, the template is
not necessary, and code can be put in source files instead of headers. (Though
it actually is templated now, not for modes, but for mocking log entries.)
Closures also make it possible to write table driven factory methods. You cannot
put classes as values into a map, but you can do it to closures.

In C++ 11, free functions ```begin()``` and ```end()``` are added to make ranged
for possible for all iterable things. Not all of them have corresponding methods
in their interface, and can be modified to have them. And overloaded operators
are sometimes outside the type. Free functions can extend types and closures make
them more powerful. I think it's time to define more functions and fewer classes.
