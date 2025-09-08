# mmwrap
A wrapper script to provide multimedia actions (alternative F-key actions) for
minimal desktop environments.

## Introduction
GNU/Linux desktop users who opt for minimal setups like tilling window managers
(i3wm, Sway, etc) often need to manually map the multimedia keys of their
keyboard to the correct command. Also, for users of DEs (GNOME, KDE, etc)
default keybindings may be incorrect or undesirable for their keyboard.

`mmwrap` is a simple wrapper bash script that invokes relevant utilities needed
for performing these multimedia functions like playing the next song, muting the
speaker or changing the display backlight intensity.

# Features
- ⏯️ Control media playback: play-pause toggle, next/prev, seek ±position
- 🔊 Control volume level and mute toggle
- 🔅 Control display backlight (brightness) level

# Installation
Download the `mmwrap` script, make it executable and move it to somewhere on
`$PATH` (e.g. `~/bin` or `~/.local/bin`):

```bash
wget https://github.com/pvonmoradi/mmwrap/mmwrap -O ~/bin/mmwrap
chmod +x ~/bin/mmwrap
```

## Dependencies
- playerctl : to control players via MPRIS D-Bus
- wpctl : to control pipewire audio server
- notify-send : to present notifications
- xbacklight | brillo | brightnessctl : to control display backlight
- dmenu | rofi | fuzzel : a generic selector menu

### Implied dependencies
- Only players controllable via
  [MPRIS](https://specifications.freedesktop.org/mpris-spec/latest/) are
  supported. Hence for mpv, [mpv-mpris](https://github.com/hoyon/mpv-mpris) is
  required.
- The notification presentation was developed and tested with
  [dunst](https://dunst-project.org/documentation/). It has a good support for
  `--icon` and features a nifty progress-bar. Other DE notification daemons may
  ignore `-t` or render a tiny picture for `-i`. We use icons to show album art
  of the playing media.
- unix tools like `awk`, `base64`, `cut`, `sed`, and `find` are assumed to be
  present on system.
  
In summary, in a Debian-like distro, a suitable set of dependencies can be
installed with:

```bash
sudo apt install playerctl pipewire-audio dunst libnotify-bin
sudo apt install brightnessctl brightness-udev # or xbacklight or brillo depending on availability
sudo apt install dmenu # or rofi or fuzzel depending on X11 or Wayland
sudo apt install mpv mpv-mpris # or vlc
```

# Usage
Three subcommands are available: `player`, `volume` and `backlight`. Check
`mmwrap -h` for description:

```console
$ mmwrap -h
A wrapper script for multimedia actions (F-keys)

USAGE:
  mmwrap [OPTIONS] SUBCOMMAND
  mmwrap player {play-pause|next|prev|previous}
  mmwrap player seek ±seconds
    Control media playback
  mmwrap volume ±percent
    Increase/decrease volume level
  mmwrap volume toggle-mute {speaker|mic|microphone}
    Toggle source/sink mute state
  mmwrap backlight ±percent
    Increase/decrease screen backlight

OPTIONS:
    -V|--version
        Print the script version
    -v|--debug
        Set log level to 'debug'
    -h|--help
        Print the help message
```

## Notes
- For `player` subcommand, when multiple players are run and at least one of
  them is in "Playing" state , `play-pause` pauses all of them. Upon calling
  `play-pause` again, a menu is presented to the user so he can select the
  player to unpause.
  In the same case, `seek` and `next`/`prev` take effect onto the **first**
  "Playing" player. Meaning if more than 1 player is "Playing", the player
  affected is decided by whichever `playerctl` finds first.

## Examples
The script can to be called in the window manager's config file or in the config
file of the key-binder daemon in a format like this:

*`bind_cmd`* [`XF86Symbol`](https://wiki.linuxquestions.org/wiki/XF86_keyboard_symbols) *`exec_cmd`* `mmwrap SUBCOMMAND ARGS`

For example, here is how the script can be used in an i3wm config file:

```config
# keyboard multimedia functions keybindings
# audio: volume adjustment (±5%)
bindsym XF86AudioRaiseVolume exec --no-startup-id mmwrap volume +5%
bindsym XF86AudioLowerVolume exec --no-startup-id mmwrap volume -5%

# audio: speaker/mic mute toggle
bindsym XF86AudioMute    exec --no-startup-id mmwrap volume toggle-mute speaker
bindsym XF86AudioMicMute exec --no-startup-id mmwrap volume toggle-mute mic

# brightness: intensity adjustment (±10%)
bindsym XF86MonBrightnessUp   exec --no-startup-id mmwrap backlight +10%
bindsym XF86MonBrightnessDown exec --no-startup-id mmwrap backlight -10%

# media control: play/pause(F9)
bindsym XF86Tools    exec --no-startup-id mmwrap player play-pause

# media control: next(F12)
bindsym XF86Explorer exec --no-startup-id mmwrap player next

# media control: prev(F11)
bindsym XF86LaunchA  exec --no-startup-id mmwrap player prev

# media control: seeking by ±5 seconds
bindsym Shift+XF86Explorer exec --no-startup-id mmwrap player seek +5
bindsym Shift+XF86LaunchA  exec --no-startup-id mmwrap player seek -5
```

## Customization
User-configurable variables are listed at the top of the script. Refer to the
comments for explanation. Modify as desired.

## Extension
The script is developed and tested on a laptop, so it assumes a single user,
single monitor and single master audio sink and source. For a different setup,
or if another selector menu app or backlight app is desired, modify
*`subcommand`*_**`driver`**_*`op`* functions.

# Development
- linter: `shellcheck`
- formatter: `shfmt -i 4 -bn -ci -sr`
