# Impetus

Linux lacks a good key remapping solution. In order to achieve satisfactory
results a medley of tools need to be employed (e.g xcape, xmodmap) with the end
result often being tethered to a specified environment (X11). keyd attempts to
solve this problem by providing a flexible system wide daemon which remaps keys
using kernel level input primitives (evdev, uinput).

# UPDATE (v2-beta)

master is currently tracking `v2-beta`. Things should be reasonably backwards
compatible but may occasionally break before v2 leaves beta. If you are looking
for something a bit more stable you may be interested the [v1](https://github.com/rvaiya/keyd/tree/v1) branch.

*NOTE: For those migrating their configs from v1, please see the
[changelog](CHANGELOG.md) for a list of changes.*

# Features

keyd has several unique features many of which are traditionally only
found in custom keyboard firmware like [QMK](https://github.com/qmk/qmk_firmware)
as well as some additional ones.

Some of the more interesting ones include:

- Layers (with support for [hybrid modifiers](man.md#modifiers)).
- Display server agnostic application remapping (Currently supports X, sway, and gnome).
- Key overloading (different behaviour on tap/hold).
- Per keyboard configuration.
- Instantaneous remapping (no flashing required).
- VT support (works everywhere).
- A [simple](#sample-config) and intuitive config format.
- First class support for modifier overloading.

### Goals

  - Speed       (a non garbage-collected input loop that takes <<1ms)
  - Simplicity  (a config format that is intuitive)
  - Correctness (modifiers that play nicely with layers by default)

### keyd is for people who:

 - Would like to experiment with custom [layers](https://beta.docs.qmk.fm/using-qmk/software-features/feature_layers) (i.e custom shift keys)
   and oneshot modifiers.
 - Like tiny daemons that adhere to the Unix philosophy.
 - Want a keyboard config format which is easy to grok.
 - Want to have multiple keyboards with different layouts on the same machine.
 - Want to put the control and escape keys where God intended.
 - Wish to be able to switch to a VT to debug something without breaking their keymap.

### What keyd isn't:

 - A tool for launching arbitrary system commands as root.
 - A tool for programming individual key up/down events.

# Dependencies

 - Your favourite C compiler
 - libudev

## Optional

 - python      (for application specific remapping)
 - python-xlib (only for X support)

# Installation

## From Source

    sudo apt-get install libudev-dev # Debian specific, install the corresponding package on your distribution

    git clone https://github.com/rvaiya/keyd
    cd keyd
    make && sudo make install
    sudo systemctl enable keyd && sudo systemctl start keyd


## Remapping (experimental)

- Add yourself to the keyd group:

	usermod -aG keyd <user>

- Populate `~/.keyd-mappings`:

E.G

	[Alacritty]

	alt.] = macro(C-g n)
	alt.[ = macro(C-g p)

	[Chromium]

	alt.[ = C-S-tab
	alt.] = macro(C-tab)

- Run:
	keyd-application-mapper


Class names can be discovered with `keyd-application-mapper -m`.

## SBC support

Experimental support for single board computers (SBCs) via usb-gadget
has been added courtesy of Giorgi Chavchanidze.

See [usb-gadget.md](src/vkbd/usb-gadget.md) for details.

## Packages

Third party packages for the some distributions also exist. If you wish to add
yours to the list please file a PR. These are kindly maintained by community
members, no personal responsibility is taken for them.

### Arch

[AUR](https://aur.archlinux.org/packages/keyd-git/) package maintained by eNV25.

# Quickstart

1. Install keyd

2. Put the following in `/etc/keyd/default.conf`:

```
[ids]

*

[main]

# Maps capslock to escape when pressed and control when held.
capslock = overload(control, esc)

# Remaps the escape key to capslock
esc = capslock
```

3. Run `sudo systemctl restart keyd` to reload the config file.

4. See the [man page](man.md) for a comprehensive list of config options.

*Note*: It is possible to render your machine unusable with a bad config file.
Should you find yourself in this position, the special key sequence
`backspace+backslash+enter` should cause keyd to terminate.

# Sample Config

	[ids]

	*
	
	[main]

	leftshift = oneshot(S)
	capslock = overload(symbols, esc)

	[symbols]

	d = ~
	f = /
	...

# Recommended config

Many users will probably not be interested in taking full advantage of keyd.
For those who seek simple quality of life improvements I can recommend the
following config:

	[ids]

	*

	[main]

	shift = oneshot(shift)
	meta = oneshot(meta)
	control = oneshot(control)

	leftalt = oneshot(alt)
	rightalt = oneshot(altgr)

	capslock = overload(C, esc)
	insert = S-insert

This overloads the capslock key to function as both escape (when tapped) and
control (when held) and remaps all modifiers to 'oneshot' keys. Thus to produce
the letter A you can now simply tap shift and then a instead of having to hold
it. Finally it remaps insert to S-insert (paste on X11).

# FAQS

## What about xmodmap/setxkbmap/*?

xmodmap and friends are display server level tools with limited functionality.
keyd is a system level solution which implements advanced features like
layering and [oneshot](https://beta.docs.qmk.fm/using-qmk/software-features/one_shot_keys)
modifiers.  While some X tools offer similar functionality I am not aware of
anything that is as flexible as keyd.

## What about [kmonad](https://github.com/kmonad/kmonad)?

keyd was written several years ago to allow me to easily experiment with
different layouts on my growing keyboard collection. At the time kmonad did not
exist and custom keyboard firmware like
[QMK](https://github.com/qmk/qmk_firmware) (which inspired keyd) was the only
way to get comparable features. I became aware of kmonad after having published
keyd. While kmonad is a fine project with similar goals, it takes a different
approach and has a different design philosophy.

Notably keyd was written entirely in C with performance and simplicitly in
mind and will likely never be as configurable as kmonad (which is extensible
in Haskell). Having said that, it supplies (in the author's opinion) the
most valuable features in less than 2000 lines of C while providing
a simple language agnostic config format.

## Why doesn't keyd implement feature X?

If you feel something is missing or find a bug you are welcome to file an issue
on github. keyd has a minimalist (but sane) design philosophy which
intentionally omits certain features (e.g execing arbitrary executables
as root). Things which already exist in custom keyboard firmware like QMK are
good candidates for inclusion.

# Contributing

See [CONTRIBUTING](CONTRIBUTING.md).
