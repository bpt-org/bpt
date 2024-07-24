# bpt - Basic/Bash Package Tooling

> Skip to [Installation](#installation) | [Philosophy](#philosophy)

Non-root package manager for Linux.

## What does bpt do?

bpt allows to install Linux programs inside the user's `$HOME`.

This comes with several disadvantages:

* Only scripts work out of the box.
* Programs require compilation for each system.
* Programs are linked against the system's libraries.
* Packages only work for the system they were built against.
* Changes on the system will break programs.
* Dependencies must be handled manually.

## Isn't this a horrible idea?

Yes, it is.

## Installation

bpt needs to be unpacked manually:
```
tar -xf bpt-any.tar.xz -C $HOME/.local/opt
```

Then chmod and run the deployment script:
```
$HOME/.local/opt/libexec/deploy
```

## Philosophy

### Directories

This is the directory tree for bpt:

```
HOME/.local/opt/bin
HOME/.local/opt/bin/bpt
HOME/.local/opt/etc
HOME/.local/opt/etc/bpt
HOME/.local/opt/etc/bpt/bpt.conf
HOME/.local/opt/etc/bpt/bpt.conf.d
HOME/.local/opt/etc/sources.list
HOME/.local/opt/var
HOME/.local/opt/var/packages
HOME/.local/opt/var/buckets
HOME/.local/opt/var/buckets/main
```

We'll refer to `HOME/.local/opt` as `ROOT`.

To avoid conflicts with XDG Base Dir Specs, every program is installed
on its own directory under `ROOT` and the binary files symlinked to `ROOT/bin`.

By including `ROOT/bin` to path, the user's personal `bin` remains free.

### Packages

bpt saves its available packages inside `ROOT/var/packages` and checks for
a matching program name on each installation.

If more than one version is available, a prompt will let the user choose.
Reinstallations and (up|down)grades are also handled by bpt.

If there are no packages matching the program name, the user will have
to build a package from a *bucket* or fetch one from a *source*.

#### Buckets

This are git repositories containing PKGBUILD files with the commands and
files for bpt to compile and package the program.

The PKGBUILD's variables and functions are a subset of Archlinux's
and bpt tries to stay compatible with them. HOWEVER, bpt WILL NEVER fetch
directly files from AUR. We don't endorse DDoSSing their servers with queries.

bpt offers a default bucket called `main` with a selection of progrmas. The name
does not have to be prefixed to install programs, unless the default bucket
is changed with `--main-bucket|-m`.

Every package built from a bucket will become available inside `ROOT/var/packages`.

### Sources

This are websites that serve ready-to-install packages, they are first downloaded
by bpt inside `ROOT/var/packages` and then become available for installation.

Keep in mind that packages are built against the system's libraries, therefore they
are specificly made for the machine they were built against.
