# Contributing

TO-DO:
* Enforce conventional Commits
* Setup a CI pipeline
* Explain Google Shell Styleguide
* Explain Scoop's directory separation

## Development

> Remember to follow the proper git guidelines at all times.

1. Create the target installation directory:
```
mkdir $HOME/.local/opt
```

2. Clone the repository on the target directory:
```
git -C $HOME/.local/opt clone https://github.com/bpt-org/bpt
```

3. Make changes on a development branch.

4. Update the PKGBUILD inside main.

> TO-DO: Create a development bucket in another branch.

5. Create a package with the new version.
```
bpt build bpt
```
