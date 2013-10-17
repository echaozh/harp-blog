I am proud to announce that [Vobile](http://vobileinc.com/) joins the 25,000
organizations using [GitLab](http://gitlab.org/). Well, sort of. I've just
managed to deploy a demo instance with Vobile Seals(R) authentication support.

Seals is Vobile's internal project management system, with all employees
automagically registered as users on employment. It is based on trac, with some
extension to manage users across trac projects.

<!-- more -->

For someone who have not touched Ruby for a several years, and who have hardly
touched Rails, to add a new authentication strategy to GitLab wasn't an easy
job.

I first aimed to add an authenticatable for
[devise](https://github.com/plataformatec/devise). I managed to find [a question
on StackOverflow
](http://stackoverflow.com/questions/14880552/getting-devise-to-recognize-my-authentication-module) and from there a link to [a post about adding a remote
authenticatable
](http://4trabes.com/2012/10/31/remote-authentication-with-devise/). I went half
way through the post, having copied all the code to my .rb files. Then I came
across [lib/gitlab/ldap/user.rb
](https://github.com/gitlabhq/gitlabhq/blob/6-1-stable/lib/gitlab/ldap/user.rb)
when I was trying to understand the mechanism how a Rails app integrates with
devise.

I was enlightened when I saw the LDAP authentication implementation. Well, it's
not an OAuth implementation. Rather, user is looked up at some remote service
instead of the database, just like the Seals authentication method in my mind.
It's a known good implementation, which works with GitLab, so I decided to stick
with it.

Instead of a completely separate authentication method which parallels LDAP,
I decided to start from something more conservative, calling Seals from LDAP,
and pretend that Seals is a special LDAP provider. This is mainly because there
are quite some `ldap_user?` calls scattered around the app code, with many other
ldap keywords. I don't want to provide a `seals_user?` predicate, or a seals
version for every other occurrence of ldap. Also, in the beginning, I am
completely in the dark as to how the config files are read and parsed, and how
the clockworks click into each other. I wasn't sure I could provide an entry
interface for the new method. Now that I am more certain about how things work,
and later I may take some time to completely separate the 2 methods.

Having written code and designed systems as part a team for some years now, my
desire to complicate things by wrapping them in layers has ceased long ago.
Code intelligibility comes first for me. Well, I have to say, the Rails
infrastructure is not beast easily tamable when you're trying desparately to
get a grip on the reins.

I ran into a problem after the authentication succeeds. The user info is looked
up and saved in database, then a 500 page is shown. The exception traceback
complains about a nil response after an `@app.call(env)` call in
`ActionDispatch::BestStandardsSupport#call()`. God, the `@app.call(env)` is in
the middle of a long chain of middleware calls. Looking back and forth, you can
see middlewares calling you and having just been called by you. Somewhere
something is wrong, and a nil response has gradually bubbled up to where the
response is used blindly and an exception is thrown. I had no idea where the
nil originates, or why, or how to fix it. After fixing it, I still have no idea.
I created a separate login page for Seals credentials, but couldn't the app
work without it? Not a clue.

I think Ruby, and especially Ruby on Rails, may have gone too far. I actually
love metaprogramming, and prefer Ruby over Python for its ability to provide
that functionality in a neat way. However, metaprogramming does downgrade the
readability of the code. When methods and their names are dynamically generated,
it is pratically impossible to tell where and how a method is defined with a
simple grep. For the writer, the code is dedup'ed and elegant; but as for the
readers, especially casual readers, it is definitely a pain.

As to the chain of middleware problem, I think it's solvable. The real problem
is that on errors, the part popped out of the call stack is the most
interesting, but cannot be presented to the debugger. To solve it, there are
actually 2 solutions:

1. always keep a whole view. Use a list of hooks instead of letting the hooks
link to each other. Rather, chain them on invocation. This is the method
employed by Emacs Lisp and JavaScript libraries solving the callback hell like
[step](https://github.com/creationix/step) and
[async.series](https://github.com/caolan/async#series);
1. use a Writer monad like mechanism to keep track of what have been called
alongside with the results returned and display them when necessary.

The hook list method is a more direct hit to the core of the problem, and more
declarative. Also, it's easier to manipulate the middleware stack, inserting
and deleting some, moving the others around.

The Writer monad is a much smaller change, and is not specific to this problem.
Actually, it can help with logging in general. For daily processing, logs
should be minimal not to overflow the disk and overwhelm interesting data;
while when error occurs, the log should be as verbose as possible to help
investigation. Some of the debug level logs may be too much for every call, but
are crucial to problem solving once there is something wrong. The logs can be
gathered without being written out. When there is a problem, the whole log
buffer can be committed to disk.
