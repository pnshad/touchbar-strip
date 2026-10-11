# touchbar-strip

A status strip for the Touch Bar on T1 MacBook Pros (2016–2017: MacBookPro13,2, 13,3, 14,2, 14,3)
running Linux with [t1bridge](https://github.com/standardagents/t1bridge). It replaces t1bridge's
built-in renderer through t1bridge's own renderer selection (`~/.config/t1bridge/renderer`) and
speaks its Touch Bar hardware IPC v1. One Python file.

```
esc | 14:22 label | ▦ agent task / last step | CPU temp+fan RAM wifi battery power | ◔☀ ◔⌨ ◔🔊 🔒
```

- **Esc** (a red key) and, with **Fn** held, Esc + F1–F12 like the stock bar.
- **Clock** in 7-segment digits, in any time zone, with a label and an optional line about the
  next full hour.
- **Agent** (optional): a coding agent's state as a square cut into one cell per session —
  green working, blue waiting for you, amber idle, grey none — plus its task and latest step.
  Built in for Claude Code; any other tool through a small script (see *Providers*).
- **Machine**: a CPU graph, CPU %, temperature with a fan that spins with the fans, RAM, Wi-Fi,
  battery (coloured toward the charge limit) and power draw. Every slot has a configurable
  **tap** and **hold**.
- **Dials**: screen and keyboard backlight and volume as ring dials. Slide to set; tap a
  backlight to go to the light sensor's suggestion, double-tap for max, hold for off; hold
  volume to mute.
- **Lock key**: tap locks. While a Touch ID request runs (sudo, the lock screen) the key shows
  a fingerprint: amber waiting, red on a miss, green on success.
- **Lock screen** (`lock.screen_sign`): `matrix` (default) — a red LOCKED decoding out of
  katakana and a green UNLOCKED when your finger is recognised; `off` — a black bar and the
  fingerprint key only.
- **OLED care** (`oled_care`, on by default): the picture drifts 1–2 px every 2 min (`orbit`)
  and goes black while the screen is off (`dark_on_dpms`; a touch wakes it for 15 s). Each can
  be turned off.
- **Refresh sign** (`refresh_sign`, off by default): a red PROCESSING then a green REFRESHED
  after a screen rain pass. Only useful with the rain pass from
  [omarchy-mac-kit-extras](https://github.com/pnshad/omarchy-mac-kit-extras).

## Install

Needs Python 3, pycairo and PyGObject (Pango) — already present on Omarchy; on Arch
`pacman -S python-cairo python-gobject`; on Debian/Ubuntu `apt install python3-cairo python3-gi
gir1.2-pango-1.0` — and a working t1bridge (`t1-touchbar.service`). A Nerd Font
(JetBrainsMono Nerd Font by default) gives the icons; without one the strip falls back to
monospace and the icons do not show.

```bash
git clone https://github.com/pnshad/touchbar-strip
cd touchbar-strip
./touchbar-strip --preview /tmp/strip.png      # one frame, no hardware needed; look at it
mkdir -p ~/.local/lib/touchbar-strip ~/.config/touchbar-strip
install -m 755 touchbar-strip ~/.local/lib/touchbar-strip/
install -m 600 config.example.json ~/.config/touchbar-strip/config.json
./touchbar-strip --check                        # what each slot will do, what is missing
ln -sf ~/.local/lib/touchbar-strip/touchbar-strip ~/.config/t1bridge/renderer
systemctl --user restart t1-touchbar             # or however your t1bridge runs the bar
```

Undo: `rm ~/.config/t1bridge/renderer` and restart t1bridge; it goes back to its own renderer.
If the strip ever exits, t1bridge falls back to its built-in renderer, so the bar never stays
dead. [omarchy-mac-kit](https://github.com/pnshad/omarchy-mac-kit) does all of this as its
`touchbar` module.

Logs: `journalctl --user -u t1-touchbar` (lines start with `touchbar-strip:`).

## Config

`~/.config/touchbar-strip/config.json` (or `$TOUCHBAR_STRIP_CONFIG`, or `--config FILE`).
Re-read every 2 s: save and the bar changes, no restart. Every key is optional; anything left
out keeps its default from [`config.example.json`](config.example.json). A config with an
error is ignored as a whole (one journal line says why) and the defaults are used.
[`examples/full.json`](examples/full.json) turns every feature on (agent with task progress,
room-loudness hint, an hour mark, every hold, the refresh sign), using the omarchy-mac-kit
helper names.

Everything decorative is optional. Want the lock key to play a short rain over the screen
before it locks (needs the `screensaver` module of
[omarchy-mac-kit-extras](https://github.com/pnshad/omarchy-mac-kit-extras))? Set `"lock": {"tap": "lock-rain", "hold": "lock"}` under `slots`:

```json
"slots": { "lock": { "tap": "lock-rain", "hold": "lock" } }
```

| Key | Default | Meaning |
|---|---|---|
| `zones` | `["clock", "agent", "machine"]` | which zones, in order; also `progress` |
| `hold_seconds` | `0.8` | how long a touch on a slot counts as a hold |
| `dial_hold_seconds`, `lock_hold_seconds` | `0.6` | the same for the dials and the lock key |
| `font` | `JetBrainsMono Nerd Font` | icon and digit font |
| `app_id_prefix` | `org.omk` | window classes the strip opens and closes: `<prefix>.hwdeck-<view>`, `<prefix>.claude` |
| `clock.timezone` | local | an IANA zone, e.g. `America/New_York` |
| `clock.label` | the zone's city | text beside the clock; `""` for none |
| `clock.hour_mark` | `null` | e.g. `"next hour in {m}m"`; `{m}` = minutes to the next full hour |
| `slots.<slot>.tap` / `.hold` | see below | an action (below) or `null` |
| `dials.hints` | `true` | the light sensor's suggested backlight as a dot on the dials |
| `dials.room_loudness` | `false` | a suggested volume from room loudness: samples the built-in mic for 1 s every 6 s, keeps one level figure, no audio |
| `lock.key` | `true` | show the lock key |
| `lock.screen_sign` | `matrix` | `matrix` or `off` (see above) |
| `oled_care.orbit` | `true` | the picture drifts 1–2 px every 2 min against burn-in |
| `oled_care.dark_on_dpms` | `true` | the bar goes black while the laptop's screen is off |
| `refresh_sign` | `false` | PROCESSING → REFRESHED after a screen rain pass (reads `helpers.refresh_signal`, a file in `$XDG_RUNTIME_DIR` the extras' `omk-rainpass` touches) |
| `agent.*` | | see *Providers* |
| `helpers.*` | | where the optional helpers live (below) |

Slots: `graph` (the CPU graph), `cpu`, `thermal`, `memory`, `wifi`, `battery`, `power`, `lock`.
A slot with a hold runs its tap when you lift the finger before the hold time; a slot with no
hold runs its tap at once.

## Actions

| Action | Does | Needs |
|---|---|---|
| `hwdeck:<view>` | opens that [hwdeck](https://github.com/pnshad/hwdeck) view full screen (`cpu`, `memory`, `fans`, `battery`, `power`, `network`, `thermal`); tap again closes it | `helpers.hwdeck`, Hyprland |
| `wall` | opens all six hwdeck views, or closes them when all are open (the `wall` module of omarchy-mac-kit-extras floats them in a 3 x 2 grid; without it they tile) | same |
| `cpu-boost` | performance power profile until held again or the CPU reaches 90 °C | power-profiles-daemon |
| `fans-max` | fans to max for a minute, then until the CPU is back to 60 °C (15 min at most) | `helpers.hw` (root helper) |
| `mem-trim` | drops clean caches and compacts memory | `helpers.hw` (root helper) |
| `wifi-toggle` | Wi-Fi radio off and on again | NetworkManager |
| `charge-limit` | battery charge limit 80 % ↔ 100 % | a writable applesmc charge knob |
| `lid-mode` | what closing the lid does: sleep ↔ stay awake | `helpers.lid` |
| `lock` | locks the session | `helpers.lock_cmd`, else `loginctl lock-session` |
| `lock-rain` | a short rain pass over the screen, then lock (an opt-in toy) | `helpers.rainpass`, else plain `lock` |
| `agent-open` | the agent zone's tap (`agent.open`) | |
| `sh:<command>` | runs `<command>` with `sh -c`, in your graphical session's environment | |
| `null` | nothing | |

An action whose helper is missing flashes **needs setup** on its slot and writes one journal
line naming what to install; an unknown action flashes **bad action**. `--check` lists every
configured action and whether what it needs is there.

The root helpers come from [omarchy-mac-kit](https://github.com/pnshad/omarchy-mac-kit)
(`fans-ram`, `smc-power`, `battery-limit`, `lid`; the rain pass for `lock-rain` from
[omarchy-mac-kit-extras](https://github.com/pnshad/omarchy-mac-kit-extras) `screensaver`), but any program with the same
interface works: `helpers.hw <fan-max|fan-auto|mem-trim>` is run as `sudo -n <hw> <cmd>` (so it
needs a NOPASSWD sudo rule for exactly those arguments), `helpers.lid toggle` prints the new mode
and `helpers.lid_state` holds it, `helpers.smc_power` is a JSON file `{"t": unix, "watts":
{"PSTR": W, ...}}` for the whole-system watts.

## Providers

The agent zone draws whatever its provider reports every 2 s:

```json
{"state": "working|waiting|idle|none", "count": 2, "task": "…", "step": "…", "progress": [3, 7]}
```

- `"provider": "claude-code"` — Claude Code sessions on this machine: `claude` processes and
  their transcripts in `~/.claude/projects/` (read locally; nothing leaves the machine). Waiting
  = the transcript ends on a question to you or a permission prompt. With `agent.projects` set
  to a folder of git repos, the newest commit there shows while no session runs.
  `"open": "tmux-attach"` makes a tap attach a terminal to the tmux session running Claude
  (or start `claude` in `agent.projects`); `"progress": true` adds the session's task list as a
  progress bar.
- `"provider": "command"`, `"command": "my-agent-status"` — any script that prints that JSON
  line (Codex, OpenCode, a CI server, …). `"open": "sh:…"` sets the tap.

Leave `agent` out of `zones` and no transcript is read and no command runs.

## Safety notes

- **`sh:` actions are code execution as you**, like any dotfile. The strip refuses a
  `config.json` (or its folder) that is not yours or is writable by group or others, and then
  uses the defaults.
- **Nothing runs while the session is locked.** Before a tap, hold or slide fires, the strip
  asks Hyprland whether the session is locked; if it is, or if that cannot be read, nothing
  runs (a hold whose action is `lock` is the one exception).
- **Touch ID: never a Cancel button, never an extra verify.** The overlay is cosmetic: it only
  reads t1bridge's `/run/t1bridge/touch-id-state.json`. A lock screen restarts a cancelled verify
  at once, and each verify is a Secure Enclave handoff; cancelling or starting verifies from a
  renderer has wedged the T1's keybag. Keep it that way if you change the code.
- Root is never used directly. The holds that need it go through a helper you installed, with a
  sudo rule for exact arguments, called via `systemd-run --user` (t1bridge's service runs with
  `NoNewPrivileges=yes`).

## Command line

```
touchbar-strip                         run (t1bridge starts it)
touchbar-strip --check [--config F]    the config in use and every action's status; exit 1 on errors
touchbar-strip --preview OUT.png [--fn] [--locked] [--lockage S] [--unlock S] [--refresh S] [--touch-id] [--zones a,b]
```

## Licence

MIT. See [CREDITS.md](CREDITS.md) for what it builds on.
