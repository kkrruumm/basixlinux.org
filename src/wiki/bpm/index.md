title: Basix Wiki

# The Basix Package Manager

dang ol build system man

bpm is the build system and package manager for Basix and is the reason the system gets to be so modular.

While there is nothing original here, I've "cherrypicked" bits and pieces of things I like about other build systems and package managers and reimplemented them in a portable and relatively small set of POSIX shell scripts.

In particular, Void's `xbps-src` was an inspiration for the format of bpm package templates.

Gentoos `portage` was the source of the "hey we should add use flags" idea.

Kiss' simplicity, decentralized nature, and philosophy was the inspiration for pretty much the rest of this.

This package manager, in its current state, is the result of ~3 years of intermittent effort, largely out of boredom. Only recently did it reach a state that I would consider usable, though. Expect rough edges, bugs, and for your computer to burst into flames.
