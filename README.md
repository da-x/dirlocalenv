About
-----

*dirlocalenv* is a small utility for allowing directory-inherited
environment variables rather than process-inherited environment
variables.

Use case
--------

Suppose you use have `project-a` that is designed to build with
`some-compiler-1.4` by default, and in addition you have `project-b`
that is designed to build with `some-compiler-1.5` by default. Now,
your `some-compiler` in `$PATH` can be only one of the two. You
could rig an wrapper script that sets up `$PATH` before entering
the environment of either projects, but you could do something else
with the help of `dirlocalenv`.

Have your homedir's bin directory `~/bin` have `some-compiler` to
the place where you cloned `dirlocalenv`. For example:

    ln -s ~/projects/dirlocalenv/dirlocalenv ~/bin/some-compiler

Now, create two files:

~/projects/project-a/.dirlocalenv:

    prepend("PATH", "/opt/some-compiler-1.5/bin")

~/projects/project-a/.dirlocalenv:

    prepend("PATH", "/opt/some-compiler-1.4/bin")

Observe that `which some-compiler` will show `~/bin/some-compiler`, 
but also observe that:

    $ cd ~/projects/project-a
    $ some-compiler --version
	Version 1.4

    $ cd ~/projects/project-b
    $ some-compiler --version
	Version 1.5

Voilà!

`dirlocalenv` works by looking up files named `.dirlocalenv` when
going up from the current directory, and Python-evaluate them in
order from top to bottom. `prepend` above should be obvious.
