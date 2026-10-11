# Credits

touchbar-strip is MIT-licensed (see [LICENSE](LICENSE)). It vendors no third-party code.

| Project | Licence | How the strip uses it |
|---|---|---|
| [t1bridge](https://github.com/standardagents/t1bridge) (and t1-revive) | MIT | the Touch Bar hardware IPC v1, the renderer hook (`~/.config/t1bridge/renderer`) and the Touch ID state file; not shipped, install it first |
| [pycairo](https://github.com/pygobject/pycairo) | LGPL-2.1 / MPL-1.1 | drawing (system package) |
| [PyGObject](https://gitlab.gnome.org/GNOME/pygobject) + Pango | LGPL-2.1+ | text layout (system package) |
| [Hyprland](https://github.com/hyprwm/Hyprland) | BSD-3-Clause | `hyprctl`: lock state, open windows, closing a window |
| [Omarchy](https://github.com/basecamp/omarchy) | MIT | `omarchy-launch-or-focus-tui`, `omarchy-system-lock` when present; nothing copied |
| brightnessctl, WirePlumber (`wpctl`), PipeWire (`pw-record`), NetworkManager (`nmcli`), power-profiles-daemon, tmux, git | MIT / MIT / MIT / GPL-2.0 / GPL-3.0 / ISC / GPL-2.0 | runtime tools, called by name |
| [hwdeck](https://github.com/pnshad/hwdeck) | MIT | the views a slot opens |
| [JetBrains Mono](https://github.com/JetBrains/JetBrainsMono) + [Nerd Fonts](https://github.com/ryanoasis/nerd-fonts) | OFL-1.1 / MIT | the default icon and digit font; configurable, not shipped |

The lock-screen sign is inspired by the digital rain of *The Matrix* (1999); no assets are used.
