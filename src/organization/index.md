# Project Organization

This project exists with the goal of being a staging area, which means to keep
as few patches as possible not upstreamed.

Therefore, vast majority of patches are intended for submission as soon as they
are ready. There are some exceptions to this - for example, our `xbps` template
is modified to use our own repositories, same with the `void-packages` config
so that building using `xbps-src` will automatically pull in binaries.

The project welcomes contributors. If you have changes, you may choose to either
work on them with us, or submit them directly for the upstream repository. In
that case, please tag the respective maintainers in order to keep track.

The current active maintainers are:

- `q66` - lead maintainer, all targets; tag always
- `pullmoll` - 64-bit big endian related matters
- `stenstorp` - 32-bit related matters

## Discussion channels

All decisions are made in our GitHub organization and there is an IRC discussion
channel on freenode, `#voidlinux-ppc`. You might want to join for quick contact
and keeping track of discussion. Since the channel does not have that much
activity, off-topic discussion is permitted, as well as discussion related to
PowerPC but not Void itself.

## Donations

The project also accepts hardware donations for testing. We do not accept
financial donations.

## Git repository structure

We have several repositories:

- `void-packages` - the fork itself; contains our "ports tree"
- `void-mklive` - also forked from upstream; contains the installer and
  live media generator
- `void-ppc-docs` - contains this documentation
- `void-ppc.github.io` - the website

There may be other repositories that are forks of various projects or the
developers' repositories.

The `void-packages` and `void-mklive` repositories have two primary branches
that are identical contents-wise. The `master` branch contains all the latest
changes and is maintained using the merge strategy, so you will always be able
to pull from it safely. The `staging` branch is like `master` but is maintained
using the rebase strategy, which means it's not safe to pull from, but has a
clean history that makes it obvious how ahead/behind upstream we are.

Other branches are usually temporary, for pull requests. Users and contributors
are encouraged to base theirs off `master` for convenience.
