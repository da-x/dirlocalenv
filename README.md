About
-----

*dirlocalenv* is a small utility for facilitating _directory-inherited_
environment variables rather than _process-inherited_ environment
variables.

Use case 1
----------

Users of Git tend to notice that the `user.email` configuration 
ends up as part of their commit meta-data. Of course, some of us
Git users have our own home projects alongside projects we do for
companies, where the E-Mail authorship should match the E-Mail
address assigned to us by the company, rather than our private 
E-Mail address.

However, it may be frustrating as Git (until a feature I helped
introducing in version 2.8) does not stop us from commiting if 
we have not set up a `user.email` config, especially for newly 
cloned repos, so new commits receives arbitrary meta-data, 
sometimes without our awareness, until it bothers other guys 
in the company upon reviewing the repo.

We can this problem by assigning an environment variable via
`dirlocalenv`.

At the top level, we maintain this default:

**~/.dirlocalenv**:

```
override("EMAIL_FOR_GIT", "private@pobox.com")
```

And at the company level, we maintain the specialization:

**`~/company/.dirlocalenv**:

```
override("EMAIL_FOR_GIT", "my.name@company.com")
```

Then, operations involving Git can receive the correct environment,
via script invocation. For example, we can use the 
following script **git-set-creds**, to set up a per-repo configuration:

```
git config user.name "My Name"
git config user.email $(dirlocalenv bash -c 'echo $EMAIL_FOR_GIT')
```

New repo initialization can be done via `git init && git-set-creds`, and
clones can also be wrapped with a similar invocation of `git-set-creds`.

Use case 2
----------

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

**~/projects/project-a/.dirlocalenv**`:

    prepend("PATH", "/opt/some-compiler-1.5/bin")

**~/projects/project-a/.dirlocalenv**:

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
