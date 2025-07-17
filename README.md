# Keepassxc-menu


## Description

Sift through keepassxc database entries and autotype the password directly in
the input form with a hotkey.

## History

This script was adapted in 2025 from the original
[keepassxc-dmenu](https://github.com/natrys/keepassxc-dmenu) to make it
compatible with wayland compositors and some general quality of life
improvements. Users of the original script should be able to migrate over by
renaming all references of `keepassxc-dmenu` to `keepassxc-menu` in their
keybinds, renaming the `kpxc_dmenu` variable in their configuration to
`kpxc_menu`, and moving the configuration file from
`~/.config/keepassxc-dmenu/config` to `~/.config/keepassxc-menu/config`.

I would like to thank @natrys for coming up with the original idea for this
script.

## Requirements

- [[https://github.com/keepassxreboot/keepassxc][keepassxc-cli]]
  (comes with default keepassxc install; minimum version requirement >= 2.5.0)
- [[https://en.wikipedia.org/wiki/Expect][Tcl + Expect]]
- Anything behaving similarly to `{d,w}menu`, like:
  - [rofi](https://github.com/davatorium/rofi] \(configuration default\)
  - [dmenu](https://tools.suckless.org/dmenu/)
  - [wmenu](https://codeberg.org/adnano/wmenu)
- Any typing application, like
  - [xdotool](https://github.com/jordansissel/xdotool) \(configuration default\)
  - [dotool](https://sr.ht/~geb/dotool/)
  - [ydotool](https://github.com/ReimuNotMoe/ydotool)
- Graphical pinentry (default is `pinentry-qt`)
- Inability to figure out the browser plugin
- Not have Mossad as enemy

## Usage

Provide a config file at `~/.config/keepassxc-menu/config` which should contain
at least `kpxc_database_path` variable. This is just a tcl file which will be
sourced. The following variables may be defined:

- `kpxc_database_path`: Path to the keepassxc database you intend to use.
- `kpxc_menu`: A program accepting input in the same manner as `dmenu` would.
- `kpxc_pinentry`: A pinentry program
- `kpxc_dotool`: A commandline for a program accepting `kpx_dotool_prefix`
  concatenated with your password on stdin
- `kpxc_dotool_prefix`: A string prefixed to your password before being sent to
  your typer of choice.
- `kpxc_notify`: A command executed right after password insertion has finished,
  intended to send you a notification.
- `kpxc_timeout`: Timeout in mins before the opened keepass database is locked

### Example configurations

#### X11

This example uses rofi for password input and xdotool to type the password
(except for the notify-send, these are the defaults if nothing is configured):

``` tcl
set kpxc_database_path $::env(HOME)/Passwords.kdbx
set kpxc_menu {rofi -dmenu -i}
set kpxc_pinentry /usr/bin/pinentry-qt
set kpx_dotool {xdotool -}
set kpx_dotool_prefix "type "
set kpx_notify "notify-send -u low \"Password inserted\" -t 1000"
# set kpxc_timeout 60
```

#### Wayland

This example uses `dotool` to type the password. Not all wayland compositors
implement the protocols used by [wtype](https://github.com/atx/wtype), which
would be preferred here, and `ydotool` does not handle keymaps different to
`us` at this time, therefore this is the current example config provided.

``` tcl
set kpxc_database_path $::env(HOME)/Passwords.kdbx
set kpxc_menu {wmenu -i -l 10 -b -p KEEPASS:}
set kpx_dotool {dotool}
set kpx_dotool_prefix "typehold 2\ntype "
set kpx_notify "notify-send -u low \"Password inserted\" -t 1000"
# set kpxc_timeout 60
```

This script is primarily meant to be invoked by pressing hotkey (either
configured in the DE, or in a hotkey daemon such as
[sxhkd](https://github.com/baskerville/sxhkd). An example for sxhkd:

``` txt
super + p
  keepassxc-menu >/dev/null 2>&1

super + alt + p
  echo exit > /tmp/keepassxc-menu/run
```

For i3/sway:

``` i3
bindsym $mod+p exec keepassxc-menu
```

For niri:

``` niri
binds {
    ...
    Mod+P { spawn "keepassxc-menu"; }
    ...
}
```
