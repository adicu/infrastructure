# Deployment

We use [gitolite](http://gitolite.com/gitolite/index.html) to manage our
deployment. Gitolite's documentation is already quite comprehensive, but
this document will give you the basics.

If you are properly authenticated, then you should be able to deploy any
of our code bases via:

```
git push git@adicu.com:<repository name>
```


## Configuration

Most of the configuration for gitolite lives in our
[gitolite-admin](https://github.com/adicu/gitolite-admin) repository,
inside the `conf/gitolite.conf` file. This file controls who gets access
to what repository, as well as what git hooks get run where.

If you are part of the `@admin` group, then you have Read, Write, and
delete access to all repository. In particular, you have `RW+` access
to the `gitolite-admin` repository (and thus have control
over who has access to the other repositories). Whenever you push the
`gitolite-admin` repository to `git@adicu.com:gitolite-admin`,
`gitolite` will automatically apply and-and-all changes you've made.


## Authentication

Authentication is controlled via `ssh`. To give someone access to a
repository, merely place their public ssh key into the `keydir`. If the
file containing their key is named `<name>.key`, then you can refer to
them in `conf/gitolite.conf` as `<name>`.


## Git Hooks

All our git hooks are placed in `local/hooks/repo-specific`. They are
wired to the appropriate repository in the `conf/gitolite.conf` via the:

```
option hook.post-receive    =   <hook name>
```

lines.
