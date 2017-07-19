[<img src="https://lnl7.github.io/nix-darwin/images/nix-darwin.png" width="200px" alt="logo" />](https://github.com/LnL7/nix-darwin)

# nix-darwin

[![Build Status](https://travis-ci.org/LnL7/nix-darwin.svg?branch=master)](https://travis-ci.org/LnL7/nix-darwin)

Nix modules for darwin, `/etc/nixos/configuration.nix` for macOS.

## Install

```bash
bash <(curl https://raw.githubusercontent.com/LnL7/nix-darwin/master/bootstrap.sh)
```

This will link the system profile to `/run/current-system`. You have to create `/run` or symlink it to `private/var/run`.
If you use a symlink, you'll probably also want to add `services.activate-system.enable = true;` to your configuration.

> NOTE: the system activation scripts don't overwrite existing etc files, so files like `/etc/bashrc` won't be used by default.
Either modify the existing file to source/import the one from `/etc/static` or remove it. Some examples:

- `mv /etc/bashrc /etc/bashrc.orig`
- `echo 'if test -e /etc/static/bashrc; then . /etc/static/bashrc; fi' | sudo tee -a /etc/bashrc`
- `echo 'if test -e /etc/static/bashrc; then . /etc/static/bashrc; fi' | tee -a ~/.bashrc`

## Updating

The bootstrap installer will configure a channel for this repository.

```bash
nix-channel --update darwin
```

## Example configuration

Check out [modules/examples](https://github.com/LnL7/nix-darwin/tree/master/modules/examples) for some example configurations.

```nix
{ pkgs, ... }:
{

  # List packages installed in system profile. To search by name, run:
  # $ nix-env -qaP | grep wget
  environment.systemPackages =
    [ pkgs.nix-repl
    ];

  # Create /etc/bashrc that loads the nix-darwin environment.
  programs.bash.enable = true;

  # Recreate /run/current-system symlink after boot.
  services.activate-system.enable = true;

}
```

## Manual Install

```bash
# install nixpkgs version, which enables libsodium support (for signed binary caches)
# this is not required if you already upgraded nix at some point
nix-env -iA nixpkgs.nix

sudo ln -s private/var/run /run

git clone git@github.com:LnL7/nix-darwin.git ~/.nix-defexpr/darwin
cp ~/.nix-defexpr/darwin/modules/examples/simple.nix ~/.nixpkgs/darwin-configuration.nix

# bootstrap build using default nix.nixPath
export NIX_PATH=darwin=$HOME/.nix-defexpr/darwin:darwin-config=$HOME/.nixpkgs/darwin-configuration.nix:$NIX_PATH

# you can also use this to rebootstrap nix-darwin in case
# darwin-rebuild is to old to activate the system.
$(nix-build '<darwin>' -A system --no-out-link)/sw/bin/darwin-rebuild build
$(nix-build '<darwin>' -A system --no-out-link)/sw/bin/darwin-rebuild switch

. /etc/static/bashrc
```

... or for `fish`:

```fish
(nix-build '<darwin>' -A system --no-out-link)/sw/bin/darwin-rebuild build
(nix-build '<darwin>' -A system --no-out-link)/sw/bin/darwin-rebuild switch
```

This will create and manage a system profile in `/run/current-system`, just like nixos.

The default `NIX_PATH` in nix-darwin will look for this repository in `~/.nix-defexpr/darwin` and for your configuration in `~/.nixpkgs/darwin-configuration.nix`.
If you want to change these you can set your own with `nix.nixPath = [ ];`.

```
$ darwin-rebuild switch
building the system configuration...
these derivations will be built:
  /nix/store/vfad6xgjzr56jcs051cg6vzch4dby92y-etc-zprofile.drv
  /nix/store/cbmkscxsz0k02ynaph5xaxm1aql0p3vq-etc.drv
  /nix/store/r5fpn177jhc16f8iyzk12gcw4pivzpbw-nixdarwin-system-16.09.drv
building path(s) ‘/nix/store/wlq89shja597ip7mrmjv7yzk2lwyh8n0-etc-zprofile’
building path(s) ‘/nix/store/m8kcm1pa5j570h3indp71a439wsh9lzq-etc’
building path(s) ‘/nix/store/l735ffcdvcvy60i8pqf6v00vx7lnm6mz-nixdarwin-system-16.09’
setting up /etc...
setting up launchd services...
writing defaults...
$
```

```
$ darwin-option services.activate-system.enable                                                                                                                                            ~/src/nix-darwin
Value:
true

Default:
false

Example:
no example

Description:
Whether to activate system at boot time.
```

## Modules

- [`environment`](https://github.com/LnL7/nix-darwin/blob/master/modules/environment)
- [`launchd.daemons`](https://github.com/LnL7/nix-darwin/blob/master/modules/launchd/launchd.nix)
- [`launchd.envVariables`](https://github.com/LnL7/nix-darwin/blob/master/modules/launchd)
- [`launchd.user.agents`](https://github.com/LnL7/nix-darwin/blob/master/modules/launchd/launchd.nix)
- [`networking`](https://github.com/LnL7/nix-darwin/blob/master/modules/networking)
- [`nix`](https://github.com/LnL7/nix-darwin/tree/master/modules/nix)
- [`programs`](https://github.com/LnL7/nix-darwin/tree/master/modules/programs)
- [`security`](https://github.com/LnL7/nix-darwin/tree/master/modules/security)
- [`services`](https://github.com/LnL7/nix-darwin/tree/master/modules/services)
- [`system.defaults`](https://github.com/LnL7/nix-darwin/tree/master/modules/system/defaults)
- [`system`](https://github.com/LnL7/nix-darwin/tree/master/modules/system)
- [`time`](https://github.com/LnL7/nix-darwin/tree/master/modules/time)
