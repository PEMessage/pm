# pm

`pm` is a wrapper for package managers. It gives you one command-line interface for all of them.

- Supports [pacman][pacman], [paru][paru], [yay][yay], [apt][apt], [dnf][dnf], [zypper][zypper], [apk][apk], [brew][brew], [scoop][scoop], and [nix][nix] (only `nix profile` and flakes, see [Nix](#nix)).
- Selects packages interactively with [fzf][fzf] and shows a package information preview.
- Is a single script. Copy it to a directory in your `$PATH`.
- Follows the POSIX standard. It runs on most systems, including [Termux][termux].

![Demo usage](demo.gif)

## Usage

Run `pm help` to print the usage:

```
Package manager wrapper (supports: paru yay pacman apt dnf zypper apk brew scoop nix)

Usage: pm <command>

Commands:
  i,  install          Interactively select packages to install.
  i,  install <pkg>... Install one or more packages.
  r,  remove           Interactively select packages to remove.
  r,  remove <pkg>...  Remove one or more packages.
  u,  upgrade          Upgrade all installed packages.
  f,  fetch            Update local package database.
  n,  info <pkg>       Print package information.
  la, list all         List all packages.
  li, list installed   List installed packages.
  sa  search all       Interactively search between all packages.
  si  search installed Interactively search between installed packages.
  w,  which            Print which package manager pm uses.
  h,  help             Print this help.
```

## Installation

### For the current user

If the `~/.local/bin` directory exists and is in your `$PATH`, run the following commands:

```sh
curl -o ~/.local/bin/pm https://raw.githubusercontent.com/jpikl/pm/refs/heads/master/pm
chmod +x ~/.local/bin/pm
```

### For all users

```sh
sudo curl -o /usr/local/bin/pm https://raw.githubusercontent.com/jpikl/pm/refs/heads/master/pm
sudo chmod +x /usr/local/bin/pm
```

### For [Termux][termux]

```sh
curl -o /data/data/com.termux/files/usr/bin/pm https://raw.githubusercontent.com/jpikl/pm/refs/heads/master/pm
chmod +x /data/data/com.termux/files/usr/bin/pm
```

## Features

### Interactive search

If you run the `install` or `remove` command without a package name, `pm` starts an interactive package search with a package information preview.

This feature needs [fzf][fzf]. Install it first: `pm install fzf`.

### AUR helpers

On Arch Linux, `pm` can install AUR helpers.

Run `pm install <helper>`. `<helper>` is `paru`, `yay`, or their binary variant (`paru-bin`, `yay-bin`).

`pm` then uses this AUR helper instead of `pacman`.

### Nix

Support for the `nix` package manager is limited on purpose: `pm` uses only the experimental `nix profile` interface and only works with flakes.

- Only `nix profile` is used. The legacy `nix-env` interface and the traditional (channel-based) Nix workflow are not supported.
- Only flakes are supported. Packages are referenced with their flake reference (e.g. `nixpkgs#hello`); plain channels and non-flake installs are not supported.
- The `nix-command` and `flakes` experimental features are required. `pm` enables them automatically, so you do not have to configure them yourself.
- The first run is slow, because nix has to evaluate every flake and build its evaluation cache. Later runs are much faster thanks to `~/.cache/nix/eval-cache-*`. While this happens, `pm` prints `Fetching packages...` to standard error so you know it is working.

```shell
PM=nix pm install nixpkgs#hello
```

## Configuration

You configure `pm` with the following environment variables.

### PM

Forces `pm` to use a specific package manager.

Options: `paru`, `yay`, `pacman`, `apt`, `dnf`, `zypper`, `apk`, `brew`, `scoop`, `nix`.

By default, `pm` detects the package manager automatically. It checks for the binaries in the order listed above.

```shell
pm install "<package>"           # Auto detect package manager
PM=pacman pm install "<package>" # Use pacman
PM=yay pm install "<package>"    # Use yay
```

### PM_SUDO

Sets the program that runs operations as root.

```shell
PM_SUDO=sudo-rs pm install "<package>" # Use alternative sudo command
PM_SUDO=doas pm install "<package>"    # Use alternative sudo command
PM_SUDO= pm install "<package>"        # Disable execution as root
```

The default value is `sudo`. If the `sudo` binary is not available, `pm` checks for alternatives (`sudo-rs`, `doas`).

Inside [Termux][termux], `pm` disables execution as root by default. Set `PM_SUDO` to change this.

### PM_COLOR

Sets color output for non-interactive commands.

Options: `auto`, `always`, `never`.

The default value is `auto`. It outputs colors only when STDOUT is a TTY.

### PM_NIX_JOBS

Controls how many flake searches run in parallel when using the `nix` package manager.

```shell
PM_NIX_JOBS=2 pm install # Limit parallel nix search to 2
```

The default value is the number of registry flakes (one worker per flake; nix search mostly waits on I/O, so running all of them at once is fine).

### PM_NIX_TIMEOUT

Controls how long a single flake search may take when using the `nix` package manager. The first evaluation of an uncached flake can take a very long time; the timeout prevents one slow flake from keeping the search (and interactive selection) open forever.

```shell
PM_NIX_TIMEOUT=180 pm install # Give slow flakes up to 3 minutes
PM_NIX_TIMEOUT=0 pm install   # Disable the timeout
```

The default value is `300` seconds.

## FAQ

### How to select multiple packages in interactive mode?

Press `TAB` to select or deselect multiple packages.

See [fzf docs](https://github.com/junegunn/fzf#using-the-finder) for more keyboard shortcuts.

### Is this better than my package manager?

Probably not. It can be more convenient in some cases:

1. If you switch between distros often, `pm` saves you from learning each package manager's command-line interface.
2. Interactive package selection helps when you search for a package and you do not know its exact name.

### Can you support package manager XYZ?

Create [an issue](https://github.com/jpikl/pm/issues) for the request. I will look into it.

## License

[apt]: https://salsa.debian.org/apt-team/apt
[apk]: https://wiki.alpinelinux.org/wiki/Alpine_Package_Keeper
[brew]: https://brew.sh
[dnf]: https://github.com/rpm-software-management/dnf
[fzf]: https://github.com/junegunn/fzf
[nix]: https://nix.dev/manual/nix/latest/command-ref/new-cli/nix3-profile
[pacman]: https://wiki.archlinux.org/title/Pacman
[paru]: https://github.com/Morganamilo/paru
[scoop]: https://scoop.sh
[termux]: https://termux.dev
[yay]: https://github.com/Jguer/yay
[zypper]: https://en.opensuse.org/Portal:Zypper
