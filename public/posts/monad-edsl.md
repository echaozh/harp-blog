DSLs are mostly standalone scripts or snippets, with a syntax different any other
general purpose languages (otherwise they're not in DSL, but just this language).
It's easy to tell what is a DSL: not C++, but can be loaded and executed by a
C++ program; and what's not: C++.

However, for EDSLs, the water is not this clear. At one extreme of the spectrum,
EDSLs in Lisp is undeniably a DSL, but embedded in Lisp. On the other end, Ruby
EDSLs are just ruby code that needs more explanation to clarify its meaning, and
looks a little bit cooler than other code.

<!-- more -->

I'm gonna talk about Ruby-ish "EDSL"s today. To give non-Haskellers a feeling of
the language, and hopefully may bring some of them to this wonderful language.

If you know Ruby, especially Ruby on Rails, you must have seen EDSLs, or just
DSLs as they're called by Rubyists. Say the query building in ActiveRecord, with
select, joins, where, group, and other keywords from SQL used in Ruby code as if
they're valid Ruby. And they are. Or a more alien looking HTML building DSL,
without dots to chain all keywords, but organized just like a HTML, or rather,
a [Jade](http://jade-lang.com/) file. Like this:

<script src="https://gist.github.com/echaozh/9348484.js"></script>

Now, doing the same in Haskell is the most widespread way to build HTMLs.
[Blaze HTML](http://jaspervdj.be/blaze/)
[has 91 reverse dependencies on hackage](http://packdeps.haskellers.com/reverse/blaze-html)
(not counting many blog sites which are not on it), while its rivals all have
fewer than 40 (and half of Hamlet's reverse dependencies start with Yesod, half
of heist's contain snap). To quote its
[reference](http://hackage.haskell.org/package/blaze-markup-0.6.0.0/docs/Text-Blaze.html):

<script src="https://gist.github.com/echaozh/9348690.js"></script>

Neat, isn't it? And it's better than Ruby EDSLs, as it's simpler than the safe
ones, and safer than the clean ones. For the Ruby snippet I show you previously,
it uses `instance_eval` to make `ul` & `li` callable from inside the block of
code. `instance_eval` is considered eval, well, evil, sorry about the typo. It
changes the context of the executing block (and then you're afraid, Ruby users?
JavaScript will scare you to death by letting you guess what `this` is inside a
function call), whatever it means. For cleaner versions, like in the
`FormHelper` from RoR, a `f.` prefixes every EDSL verb, and making it less
cryptic, and more Ruby like.

The Haskell version is possible thanks to something called
"[Monads](http://en.wikipedia.org/wiki/Monad_(functional_programming))". And
thanks to Haskell's
[do notation](http://www.haskell.org/haskellwiki/Monad#Special_notation) to
avoid strange arrow like symbols (`>>=`). Reading the latter link, you can find
out how Haskellers think of monads: as a _composable_ computation descriptions.
Well, I always find this line vague, so let's just say it's an evaluation
context, like the `this` or `self` in various mainstream languages, in a very
stretched sense. Say in the example below:

<script src="https://gist.github.com/echaozh/9349531.js"></script>

You can save it in `stack.hs`, and run it with `runghc stack.hs`. It prints
`[2,1]`, as you have seen, with 1, 2, and 3 pushed one after another, and the
last number popped. The `testStack` has a type of `Stack Int ()`, which is its
return type, making it a constant with no arguments. Wait, it's a constant? but
it's all actions inside, ow can it be a constant? Say I am writing in Ruby or
C++, can I have a constant with 3 push statements and 1 pop? I can't even have
a variable with such a value, which is not even a value. `testStack` doesn't
make a stack and return it, as you can see from the code, it returns a string
saying it's not a stack builder!

Well, keep reading the HaskellWiki, you will find:

> The essence of monad is thus separation of composition timeline from the
> composed computation's execution timeline, as well as the ability of
> computation to implicitly carry extra data, as pertaining to the computation
> itself, in addition to its one (hence the name) output, that it will produce
> when run (or queried, or called upon).

In mortal language, it basically means 2 things:

1. calling `testStack` doesn't run it immediately. Given a stack, it can manipulate it and return a human readable explanation, but it doesn't produce a stack by itself. It's like a [promise (as in concurrent programming)](http://en.wikipedia.org/wiki/Futures_and_promises) of stack operations, but pending and not resolved;
2. when you actually goes on to resolve the pending promise, with a real stack, by calling `execState`, the operations will happen, and the string will be returned. And the stack magically makes its appearance in the `Stack` monad to `testStack`.

Now, if you look at `State`'s definition, it's
`newtype State s a = {runState :: s -> (a, s)}`, meaning it's a "structure"
with 1 field called `runState`, which is a function type, taking a `s` (or
`[Int]` in our example), and returns `(a, s)`, which is a tuple
(`(String, [Int])` in `testStack`'s case). This function is the "computation
description" mentioned in HaskellWiki: `push` pushes something to the stack,
and `pop` pops something out. The structure is thus a Monad.

Though the stack operations are simplistic, they look like Ruby EDSLs, or even
better. Unlike Ruby, the EDSL can be easily extended, even in other modules, or
even other projects. You don't need to inject or monkey patch anything, you just
define a function returning a `Stack` monad. Actually, the `testStack` function,
despite its name and the useless deed it does, is a valid verb for stack
operation. You can define `pushTwice` or `popTillEmpty` or various other
interesting operations.

For serious, useful and understandable monad starters, you can take a look at
[parsec](http://legacy.cs.uu.nl/daan/parsec.html). `Maybe` is a monad with no
EDSL like verbs, but very useful. You can consider it as a safe nullable pointer,
which will never throw a `NullPointerException` in your face. `Either` is also
interesting, as it makes skipping computation on exception possible without
`goto` or stack unwinding. Lists (`[]`) are monads as well, and
[list comprehension is just a syntax sugar for monadic operations](http://learnyouahaskell.com/a-fistful-of-monads).
The day I first ran into this pearl was made awesome by it.
