# bpt - Basic/Bash Package Tooling

> Skip to [Installation](#installation) | [Philosophy](#philosophy)

Non-root package manager for Linux.

## What does bpt do?

bpt allows to compile, package, and install Linux programs inside the user's `$HOME`.

## Installation

1. Download de latest release from GitHub to your `$HOME`.

> e.g. for the release X.Y.Z

```
curl -L -O https://github.com/bpt-org/bpt/releases/download/vX.Y.Z/bpt-any.tar.xz
```

2. Unpackage it on the target directory defined below:
```
mkdir -p $HOME/.local/opt && tar -xf bpt-any.tar.xz -C $HOME/.local/opt
```

3. Initialize the program:
```
$HOME/.local/opt/bpt/bin/bpt init
```

4. Update your bash environment:
```
source ~/.bashrc
```

5. Check the program's help:
```
bpt help
```

## Philosophy

### Directories

bpt organizes its directories under `$HOME/.local` to follow the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) about dotfiles.

To avoid colliding with XDG, bpt mimics the [`/opt`](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s13.html) directory structure of the Filesystem Hierarchy Standard.

```
$HOME/.local
.
├── bin
│   └── ...
├── opt
│   ├── bin
│   ├── bpt
│   ├── etc
│   │   └── bpt
│   │       ├── bpt.conf
│   │       ├── buckets.list
│   │       └── sources.list
│   └── var
│       ├── buckets
│       └── packages
│           ├── ...
│           └── main
├── share
│   └── ...
└── state
    └── ...

```

The internal bpt directory structure mimics the one used by [`scoop`](https://github.com/ScoopInstaller/Scoop):

* `bin`: executables that will be available in PATH.
* `libexec`: subcommands that will be called from `bpt`.
* `lib`: functions that execute the actions of the subcommands.

Some important points:

* Every subcommand in `libexec` has a counterpart file inside `lib`.
* Some `lib` files are auxiliar functions and have no counterpart inside `libexec`.
* Files inside `libexec` are executed by bash, therefore have execution permissions.
* Files inside `lib` are sourced by bash, therefore do not have execution permissions.

Configurable files:

* `buckets.list`: buckets whose url is known by bpt and will be autofilled.
* `sources.list`: sources where bpt can pull ready-to-install packages.
* `bpt.conf`: settings for the users to tweak the bpt behaviour.


Every new program will be installed on `$HOME/.local/opt` inside its own folder, for example `curl` side by side with `bpt`:

```
$HOME/.local/opt
.
├── bin
│   ├── bpt -> $HOME/.local/opt/bpt/bin/bpt
│   ├── curl -> $HOME/.local/opt/curl/bin/curl
│   └── curl-config -> $HOME/.local/opt/curl/bin/curl-config
├── bpt
│   ├── bin
│   │   └── bpt
│   ├── lib
│   │   └── ...
│   └── libexec
│       └── ...
└── curl
    ├── bin
    │   ├── curl
    │   └── curl-config
    ├── include
    │   └── ...
    ├── lib
    │   └── ...
    └── share
        └── ...

```

Executable files are symlinked to `$HOME/.local/opt/bin`, which is added to PATH with `.bashrc`.


* `bin`

### Packages

The Bpt Build System (BPS) is inspired on the [Archlinux Build System](https://wiki.archlinux.org/title/Arch_build_system).
By making use of [PKGBUILD](https://wiki.archlinux.org/title/PKGBUILD) templates bpt is capable
of building from source packages compatible with the target system. It's a current WIP trying to obtain
a 1:1 compatibility on the template functions and variables.

Since every package will need to be fine-tuned to the system's provided libraries and dependencies,
a distribution system for PKBUILD files called *buckets* is implemented.

#### Buckets

bpt buckets are inspired on [scoop's buckets](https://github.com/ScoopInstaller/Scoop/wiki/Buckets), a git repository
of compatible PKGBUILD files for the host system. This distribution model was prefered over [AUR](https://wiki.archlinux.org/title/Arch_User_Repository)
because it allows to have multiple PKGBUILD variations maintained separately, and also different collection's to
satisfy the machine's users needs.

##### Known buckets

The following buckets are known to bpt:

* [main](https://github.com/bpt-org/main) - Default bucket with important CLI programs.
* [anakena](https://github.com/tvillega/dcc-tools/tree/bpt) - Official bucket for Universidad de Chile's DCC server.

We don't endorse the automatized pulling of PKGBUILD from Archlinux servers.

## Contributing

Please refer to the [contrib file](CONTRIBUTING.md).
