# bpt - Basic/Bash Package Tooling

> Skip to [Installation](#installation) | [Usage](#usage) | [Philosophy](#philosophy) | [Personal Comments](#personal-comments)

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

## Usage

### Help

* You can check the list of available commands:

```
bpt help
---
bpt - Basic/Bash Package Tool

  Available Commands:
    remove           Remove a program                        
    help             Prints this help message                
    list             List currently installed programs       
    build            Create a package from a PKGBUILD        
    init             Initialize files and directories        
    fetch            Download a package from a source        
    bucket           Manage bpt buckets                      
    install          Install a saved package   
```

* To see help for individual commands:

```
bpt help build
---
 bpt build <bucket>/<program>

 The packages provided by apt|rpm|pacman are built on a clean environment
 to prevent missing dependencies when installed on the target systems.
 This way the package manager fills the missing libraries by requiring
 the installation of secondary programs, called dependencies.
 
 bpt builds packages directly on the host system, therefore it depends
 on the libraries available there. This allow to quickly generate packages
 ready for use with the downside of not being portable to other machines.
```

### Build and Install

bpt needs to build the packages before installing them. It has a list of
PKGBUILD files inside the default bucket `main` with the instructions
required to automatize this process.

> Note: Currently only available `curl` and `libarchive`.

* To build the program `curl`:
```
bpt build curl
```

This will create a package compatible with bpt for the current Linux machine.

* To install the `curl` package built on the previous step:
```
bpt install curl
```

This will search for a built package and install it.
If more than one version is available, a selector will appear.

* To remove the program:
```
bpt remove curl
```

The program name must be given, as not all executables match the program name.

* To list all **executables** available via bpt:
```
bpt list
```

For instance, libarchive installs multiple binary files (none of them named 'libarchive').

### Buckets

* To obtain more PKBUILTD's than the ones provided by the `main` bucket:

```
bpt bucket add <bucket>
```

There's currently a bucket called `anakena` available for Universidad de Chile,
let's try to install the program `dcc-tools` from there:

1. Add the bucket:
```
bpt bucket add anakena
```

You don't need to provide an url for anakena since it is listed as a known
bucket inside `$HOME/.local/opt/etc/bpt/buckets.list`.

2. Build the program:
```
bpt build anakena/dcc-tools
```

You need to prefix the bucket since it is different from `main`.

3. Install the program:
```
bpt install anakena/dcc-tools
```

You need to prefix the bucket, this limitation was added to improve
the downloading and serving of packages via Apache User Directories.

4. To list all buckets added:
```
bpt bucket list
```

5. The bucket can be removed as well:
```
bpt bucket remove anakena
```

This will not remove the installed programs.

> Note: The bucket command aims to provide feature parity with scoop.

### Distribute and Fetch

bpt reads the file `$HOME/.local/opt/etc/bpt/bpt.conf` from where the
destination directory of the packages can be changed. This way a user can use
it's personal Apache User Directory to directly serve its built packages
to other users on the shared Linux machine.

To fetch a built package of dcc-tools:
```
bpt fetch anakena/dcc-tools
```

A match for `anakena` will be done inside `$HOME/.local/opt/etc/bpt/sources.list`,
a repository served by the user tvillega in a server of Universidad de Chile.

This will download the package to the packages directory.
The bucket `anakena` will be added if it wasn't added before
(because it is a known bucket). When the package has succesfuly
been downloaded, a prompt will ask the user to proceed with installation.

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

## Personal Comments

Despite all my efforts to provide a stable and consistent program,
bash was numerous quirks and some programming choices are somewhat hacky.

For example, when a coreutils program fails to parse a string or has zero matches,
it returns an empty string. In many locations the functions interpret the empty string
as a failure, but sometimes the output is an error message and it feeds a bash array
with garbage values than then make a snowball of issues.

I tried to use exit codes, but everything became too verbose and overengineered.
A revision must be done on this to achieve a good balance between stability and sanity.

Bash arrays are a godsent, I no longer need to parse `ls` output or glob expansion,
storing `find` searches on arrays instead. I give all the credits of this program
to bash arrays, that's why bpt needs bash 4 or later to work.

Another pivotal design choice was the scoop directory structure. The original bpt had
almost 500 lines of codes and it was slowly becoming inhuman to develop.

I code in `nano` (with bash highlights on), enabling the shared buffer with the `-F` option
to open multiple files and share the clipboard, and I also use the `-c` option to
have the columns/rows indicator on the bottom center.

I complement the afromented workflow by working inside a `tmux` session, this way I can test
the program, code it, and commit on different virtual windows. (I don't necesarily code
on the same machine Linux is running, therefore this works everywhere where there's ssh and a keyboard).

bpt is a mix of Scoop, Apt (the sources.list), Archlinux, XDG, an FHS concepts, so it can be a little hard
to grasp at first. However, I promise consistency on the choices, and if you are familiar
with one you'll understand the whole thing fairly quickly.

I don't document `buckets.list` and `sources.list` since they are currently a WIP and
they don't enforce much at the moment (I just match names with URL's in a hacky, unstable way).

And I thank my cat for keeping my lap warm. 3 AM is a really cold time of the day.

## Contributing

Please refer to the [contrib file](CONTRIBUTING.md).
