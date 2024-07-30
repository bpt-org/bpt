# bpt - Basic/Bash Package Tooling

> Skip to [Installation](#installation) | [Philosophy](#philosophy)

Non-root package manager for Linux.

## What does bpt do?

bpt allows to compile, package, and install Linux programs inside the user's `$HOME`.

## Installation

1. Download de latest release from GitHub.

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

Every new program will be installed on `$HOME/.local/opt` inside its own folder, for example `curl`:

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
